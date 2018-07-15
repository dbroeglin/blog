---
layout: post
comments: true
title: Analyser des paquets ISUP avec Wireshark
accroche: Il y a quelques temps j'ai eu à analyser des journaux contenant des messages ISUP avec Wireshark. L'opération n'est pas triviale, mais une fois comprise elle est relativement simple à mettre en oeuvre dans un script. Le suite de cette article décrit les étapes nécessaires à la transformation d'un message MTP en un message SIGTRAN pour permettre son analyse par Wireshark.
categories:
 - Ruby
 - Protocol
 - SS7
---
_{{ page.accroche }}_

Le décor
--------

[Wireshark](http://www.wireshark.org/) est sans aucun doute le plus connu et le plus complet des outils d'analyse de trafic libres. Aussi, lorsque j'ai eu à trouver un outil pour analyser du trafic ISUP, je me suis tout de suite tourné vers lui. Effectivement, Wireshark peut analyser des messages ISUP, l'ennui c'est que Wireshark analyse du SIGTRAN (SS7 over IP) et les traces dont je dispose sont des _dumps_ textes dans des fichiers de journaux. J'ai bien failli me plonger dans le standard et écrire mon propre analyseur ISUP (ouch...) quand j'ai découvert un petit outil fourni avec Wireshark: _text2pcap_. 

La page _man_ de _text2pcap_ indique qu'il s'agit d'un outil capable de transformer du texte (proche du format d'un _dump_ héxadécimal) en un fichier PCAP qui est le format de capture de trafic réseau utilisé en entrée par Wireshark. Moyennant une transformation des _dumps_ de l'application je peux donc facilement générer une des traces lisibles par Wireshark. L'ennui c'est que l'application fonctionne sur un réseau TDM alors que Wireshark attend des traces SIGTRAN. Il faut donc extraire les messages MTP niveau 3 et les encapsuler dans du MUA3 avant de les transformer avec _text2pcap_. L'article surivant m'a été très utile afin de comprendre suffisamment l'encodage MTP et ISUP pour faire cette transformation : [Decoding MTP 3 and ISUP](http://blog.corelatus.com/Decoding_MTP_3_and_ISUP.html). L'option `-S` de _text2pcap_, qui permet de générer des entêtes SCTP pour chaque paquet a fait le reste.

Acte premier
------------

Le message ci-dessous représente un paquet IAM (_Initial Address Message_) encapsulé dans son message MTP niveau 2 et 3. Pour simplifier nous partirons du principe que les messages sont présentés dans les journaux sous la forme suivante:

{% highlight none %}
0000  c0 c6 0b 85 e4 c8 6a 03 21 00 01 00 00 00 0a 03
0010  02 09 07 83 90 21 43 65 87 09 0a 07 83 13 21 43
0020  65 07 00 13 02 33 35 28 07 83 04 21 43 65 17 00
0030  0b 07 83 14 21 43 65 17 00 00 08 54
{% endhighlight none %}

Le script Ruby, ci-dessous, prend sur son entrée standard des paquets dans ce format et les affiche sur sa sortie standard transformés en paquets M3UA (_Signaling System 7 (SS7) Message Transfer Part 3 (MTP3) - User Adaptation Layer_) dans le format attendu en entrée par _text2pcap_.

{% highlight none ruby %}
MTP_PREFIX = %w{01 00 01 01}
PROTO_DATA = %w{00 02}

def full_length(len)
  ["00", "00", "00", "%0.2X" % len]
end

def mtp3_length(len)
  ["00", "%0.2X" % len]
end

def create_pcap_text(data)
  data = data.slice(3, data.size - 3)
  mtp3_length = data.size
  full_length = mtp3_length + MTP_PREFIX.size + 
    PROTO_DATA.size + 4 + 2 # 4 = length size, 2 = mtp3 size
  padding_length = 4 - full_length % 4 
  padding_length = 0 if padding_length == 4
  full_length += padding_length
  MTP_PREFIX + full_length(full_length) + 
    PROTO_DATA + mtp3_length(mtp3_length + PROTO_DATA.size + 2) + 
    data + ["00"] * padding_length
end

data = []
$stdin.each_line { |line|
  case line.chomp
  when /^S:([^:]+):$/ then data = []
  when /^[0-9a-fA-F]{4}  ([ 0-9a-fA-F]+)$/ then data.concat $1.split(' ')
  when /^\t.*$/, /^$/ then # ignore
  else
    $stderr.puts "Unrecognized log line: '#{line}'"
  end
}
$stdout.puts "0000  #{create_pcap_text(data).join(" ")}"
{% endhighlight none %}

Acte deuxième
-------------

Le résultat de l'exécution du script sur le paquet ci-dessus donne la ligne suivante :

`0000  01 00 01 01 00 00 00 48 00 02 00 3D 85 e4 c9 6d 13 21 00 01 00 00 00 0a 03 02 09 07 83 90 41 54 34 50 00 0a 07 83 13 79 70 45 86 06 13 02 33 35 28 07 83 04 81 73 45 02 03 0b 07 83 14 81 73 45 02 03 00 08 54 00 00 00`

qui peut alors être injectée à la commande `text2pcap -S 30,40,3 - /tmp/packet.pcap`. 

L'exécution de la commande précédente produit la sortie :

{% highlight none %}
Input from: Standard input
Output to: /tmp/packet.pcap
Generate dummy Ethernet header: Protocol: 0x800
Generate dummy IP header: Protocol: 132
Generate dummy SCTP header: Source port: 30. Dest port: 40. Tag: 0
Generate dummy DATA chunk header: TSN: 0. SID: 0. SSN: 0. PPID: 3
Wrote packet of 72 bytes at 0
Read 1 potential packet, wrote 1 packet
{% endhighlight none %}

Comme indiqué par la sortie de la commande _text2pcap_ le paramètre `-S 30,40,3` a généré les entêtes Ethernet, IP et SCTP pour encapsuler le paquet M3UA construit à l'étape précédente. Le contenu de ces entêtes n'a, pour le plupart, pas d'importance dans la suite et sont choisis arbitrairement par _text2pcap_.

Acte troisième
--------------

Finalement, il suffit de lire cette capture avec Wireshark (la version utilisée ici est la 1.2.9) ou sont équivalent en ligne de commande _tshark_. L'exécution de la commande `tshark -V -r /tmp/packet.pcap` produit :

{% highlight none %}
Frame 1 (134 bytes on wire, 134 bytes captured)
    Arrival Time: Jun 22, 2010 06:33:52.000000000
    [Time delta from previous captured frame: 0.000000000 seconds]
    [Time delta from previous displayed frame: 0.000000000 seconds]
    [Time since reference or first frame: 0.000000000 seconds]
    Frame Number: 1
    Frame Length: 134 bytes
    Capture Length: 134 bytes
    [Frame is marked: False]
    [Protocols in frame: eth:ip:sctp:m3ua:mtp3:isup]
Ethernet II, Src: 01:01:01:01:01:01 (01:01:01:01:01:01), Dst: 02:02:02:02:02:02 
[...]
Internet Protocol, Src: 1.1.1.1 (1.1.1.1), Dst: 2.2.2.2 (2.2.2.2)
[...]
Stream Control Transmission Protocol, Src Port: 30 (30), Dst Port: 40 (40)
[...]
        Payload protocol identifier: M3UA (3)
MTP 3 User Adaptation Layer
    Version: Release 1 (1)
    Reserved: 0x00
    Message class: Transfer messages (1)
    Message type: Payload data (DATA) (1)
    Message length: 72
    Protocol data 1 (SS7 message of 57 bytes)
        Parameter Tag: Protocol data 1 (2)
        Parameter length: 61
        Padding: 000000
Message Transfer Part Level 3
    Service information octet
        10.. .... = Network indicator: National network (0x02)
        ..00 .... = Spare: 0x00
        .... 0101 = Service indicator: ISUP (0x05)
    Routing label
        .... .... .... .... ..00 1000 1110 0100 = DPC: 2276
        .... 0011 0110 1010 11.. .... .... .... = OPC: 3499
        0000 .... .... .... .... .... .... .... = Signalling Link Selector: 0
ISDN User Part
    CIC: 33
    Message type: Initial address (1)
    Nature of Connection Indicators: 0x0
        Mandatory Parameter: 6 (Nature of connection indicators)
        .... ..00 = Satellite Indicator: No Satellite circuit in connection (0x00)
        .... 00.. = Continuity Check Indicator: Continuity check not required (0x00)
        ...0 .... = Echo Control Device Indicator: Echo control device not included
    Forward Call Indicators: 0x0
        Mandatory Parameter: 7 (Forward call indicators)
        .... ...0 .... .... = National/international call indicator: Call to be treated as national call
        .... .00. .... .... = End-to-end method indicator: No End-to-end method available (only link-by-link method available) (0x0000)
        .... 0... .... .... = Interworking indicator: no interworking encountered (No.7 signalling all the way)
        ...0 .... .... .... = End-to-end information indicator: no end-to-end information available
        ..0. .... .... .... = ISDN user part indicator: ISDN user part not used all the way
        00.. .... .... .... = ISDN user part preference indicator: ISDN user part preferred all the way (0x0000)
        .... .... .... ...0 = ISDN access indicator: originating access non-ISDN
        .... .... .... .00. = SCCP method indicator: No indication (0x0000)
        .... .... ...0 .... = Ported number translation indicator: number not translated
        .... .... ..0. .... = Query on Release attempt indicator: no QoR routing attempt in progress
    Calling Party's category: 0xa (ordinary calling subscriber)
        Mandatory Parameter: 9 (Calling party's category)
        Calling Party's category: ordinary calling subscriber (0x0a)
    Transmission medium requirement: 3 (3.1 kHz audio)
        Mandatory Parameter: 2 (Transmission medium requirement)
        Transmission medium requirement: 3.1 kHz audio (3)
    Called Party Number: 123456789
        Mandatory Parameter: 4 (Called party number)
        Pointer to Parameter: 2
        Parameter length: 7
        1... .... = Odd/even indicator: odd number of address signals
        .000 0011 = Nature of address indicator: national (significant) number (3)
        1... .... = INN indicator: routing to internal network number not allowed
        .001 .... = Numbering plan indicator: ISDN (Telephony) numbering plan (1)
        Called Party Number: 123456789
            .... 0001 = Address signal digit: 1 (1)
            0010 .... = Address signal digit: 2 (2)
            .... 0011 = Address signal digit: 3 (3)
            0100 .... = Address signal digit: 4 (4)
            .... 0101 = Address signal digit: 5 (5)
            0110 .... = Address signal digit: 6 (6)
            .... 0111 = Address signal digit: 7 (7)
            1000 .... = Address signal digit: 8 (8)
            .... 1001 = Address signal digit: 9 (9)
            E.164 Called party number digits: 123456789
    Pointer to start of optional part: 9
    Calling Party Number: 123456700
        Optional Parameter: 10 (Calling party number)
        Parameter length: 7
        1... .... = Odd/even indicator: odd number of address signals
        .000 0011 = Nature of address indicator: national (significant) number (3)
        0... .... = NI indicator: complete
        .001 .... = Numbering plan indicator: ISDN (Telephony) numbering plan (1)
        .... 00.. = Address presentation restricted indicator: presentation allowed (0)
        .... ..11 = Screening indicator: network provided (3)
        Calling Party Number: 123456700
            .... 0001 = Address signal digit: 1 (1)
            0010 .... = Address signal digit: 2 (2)
            .... 0011 = Address signal digit: 3 (3)
            0100 .... = Address signal digit: 4 (4)
            .... 0101 = Address signal digit: 5 (5)
            0110 .... = Address signal digit: 6 (6)
            .... 0111 = Address signal digit: 7 (7)
            0000 .... = Address signal digit: 0 (0)
            .... 0000 = Address signal digit: 0 (0)
            E.164 Calling party number digits: 123456700
    Redirection Information
        Optional Parameter: 19 (Redirection information)
        Parameter length: 2
        .... .011 .... .... = Redirection indicator: call diverted (3)
        0011 .... .... .... = Original redirection reason: unconditional (national use) (3)
        .... .... .... .101 = Redirection counter: 5
        .... .... 0011 .... = Redirection reason: unconditional (national use) (3)
    Original Called Number: 123456710
        Optional Parameter: 40 (Original called number)
        Parameter length: 7
        1... .... = Odd/even indicator: odd number of address signals
        .000 0011 = Nature of address indicator: national (significant) number (3)
        .000 .... = Numbering plan indicator: Unknown (0)
        .... 01.. = Address presentation restricted indicator: presentation restricted (1)
        Original Called Number: 123456710
            .... 0001 = Address signal digit: 1 (1)
            0010 .... = Address signal digit: 2 (2)
            .... 0011 = Address signal digit: 3 (3)
            0100 .... = Address signal digit: 4 (4)
            .... 0101 = Address signal digit: 5 (5)
            0110 .... = Address signal digit: 6 (6)
            .... 0111 = Address signal digit: 7 (7)
            0001 .... = Address signal digit: 1 (1)
            .... 0000 = Address signal digit: 0 (0)
    Redirecting Number: 123456710
        Optional Parameter: 11 (Redirecting number)
        Parameter length: 7
        1... .... = Odd/even indicator: odd number of address signals
        .000 0011 = Nature of address indicator: national (significant) number (3)
        .001 .... = Numbering plan indicator: ISDN (Telephony) numbering plan (1)
        .... 01.. = Address presentation restricted indicator: presentation restricted (1)
        Redirecting Number: 12345671
            .... 0001 = Address signal digit: 1 (1)
            0010 .... = Address signal digit: 2 (2)
            .... 0011 = Address signal digit: 3 (3)
            0100 .... = Address signal digit: 4 (4)
            .... 0101 = Address signal digit: 5 (5)
            0110 .... = Address signal digit: 6 (6)
            .... 0111 = Address signal digit: 7 (7)
            0001 .... = Address signal digit: 1 (1)
            .... 0000 = Address signal digit: 0 (0)
            ISUP Redirecting Number: 123456710
    End of optional parameters (0)
{% endhighlight none %}

Les parties non pertinentes ont été remplacées par des `[...]`. Le _Payload protocol identifier_ de l'entête SCTP à pour valeur _3_ ce qui indique que le contenu du paquet est un paquet M3UA. La valeur 3 provient du paramètre `-S30,40,3` qui défini une partie du contenu de l'entête SCTP du paquet.

Epilogue
--------

Cet article passe très rapidement en revue les points clés nécessaires à l'analyse des paquets ISUP au format texte. Même si les scripts et les commandes présentées ci-dessus sont fonctionnels, il manque un certain nombre de finitions pour une utilisation intensive.  La [documentation du décodeur ISUP](http://wiki.wireshark.org/Protocols/isup) de Wireshark donne la liste des options disponibles lors de l'analyse des paquets. Le détail des possibilités de _text2pcap_ est décrit dans sa [page man](http://www.wireshark.org/docs/man-pages/text2pcap.html). A titre d'exemple, il est possible d'injecter dans le fichier PCAP généré l'horodatage exact de chaque paquet ce qui peut aider à la lecture d'une trace d'une grande taille.

Pour terminer, le lecteur qui souhaite approfondir le sujet peut trouver les standards de l'ITU à l'adresse [http://www.itu.int/rec/T-REC-Q/fr](http://www.itu.int/rec/T-REC-Q/fr). M3UA est défini par la [RFC 4666](http://tools.ietf.org/html/rfc4666). Pour une introduction complète aux différents protocoles liés à SS7 je conseille l'excellent manuel de Travis Russel [Signaling system #7](http://books.google.fr/books?id=tBHlc0J7ZAAC&lpg=PP1&dq=signaling%20system%20%237%20travis&pg=PP1#v=onepage&q&f=false).

