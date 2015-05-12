Architectures Distribuées  Partie 1 UCE Applicative

I. Objectifs 
L’objectif de cette UCE applicative est de réaliser une application Java de traduction (langage naturel oral français vers langage naturel oral anglais). Vous aurez à assembler des composants hétérogènes (qui vous seront fournis) en utilisant les technologies CORBA, EJB3, Servlet et RMI. 

II. Vue d’ensemble 
L’application comportera cinq éléments qui interagiront : 
- L’application client, en Java, qui dialogue avec une Servlet central - Une Servlet de centralisation faisant la relation entre les différents services - Un serveur de reconnaissance de la parole (RAP), relié à la Servlet centrale via CORBA - Un serveur de traduction en EJB3, relié à la Servlet centrale via SOAP - Un serveur de synthèse vocale (TTS) sous la forme d’une Servlet, relié à la Servlet centrale avec HTTP  


Le déroulement typique d’une opération de traduction sera : 
- Le client fait l’acquisition du signal sonore (parole) - Le client envoie le signal sonore encodé à la Servlet centrale en HTTP POST - La Servlet centrale Envoie du signal sonore encodé au serveur de RAP via CORBA et récupère le texte correspondant - La Servlet centrale envoie le texte au serveur de traduction via EJB3 et récupère la traduction - La Servlet centrale envoie la traduction au serveur de TTS via HTTP POST et récupère le signal sonore correspondant - La Servlet centrale envoi au client le signal sonore en réponse à sa requête HTTP - Le client récupère le signal sonore envoyé par la Servlet et le joue 

III. Serveur de RAP et SpeerProxy 
Le serveur de RAP que vous allez écrire en Java et encapsulera le composant fourni SpeerProxy, qui vous masque la complexité du décodage. Il communiquera avec la Servlet centrale via CORBA. 
SpeerProxy dispose d’une méthode publique statique : 
public static String decode(short[] audio) 
Cette méthode prend le flux audio en entrée, sous la forme d’un tableau de short. Le signal est quand à lui un waveform monocanal à 16000Hz codé en entiers de 16 bits signés (short). Elle retourne la chaîne de caractères correspondant plus ou moins à ce qui a été dit. 

IV. Serveur de traduction et TradProxy 
De même, le serveur de traduction que vous allez produire encapsulera TradProxy. Il s’agira d’un EJB3 déployé sur le serveur Glassfish de votre machine. 
TradProxy possède une méthodes publique statique : 
public static String translateFrToEn(String phrase) 
Cette méthode prend une chaîne de caractères en français, et retourne sa traduction en anglais. 

V. Serveur Text to Speech et TTSProxy 
Le serveur de TTS que vous développerez sera une Servlet qui encapsulera TTSProxy, qui se charge de transformer un texte en signal sonore. Elle sera déployée sur le serveur Glassfish de votre machine. 
TTSProxy possède deux méthodes publiques statiques : 
public static short[] synthesizeEn(String sentence) 
Cette méthode prend en entrée une phrase sous la forme d’un String java et retourne un tableau de d’entiers 16 bits signas (shorts en java), contenant le signal synthétisé à partir de la phrase et encodé sous la forme d’un waveform monocanal à 16000Hz. 

VI. Servlet centrale 
La Servlet centrale sert à masquer aux clients la complexité et l’hétérogénéité du processus de traduction d’un signal de parole en Français en un signal de parole en Anglais. Il offre aux clients une 
interface simple reposant sur HTTP. Le signal en Français est envoyé à cette Servlet sous la forme d’un paramètre POST et elle retourne le signal traduit dans le corps de la réponse HTTP. 

VII. Le client Java 
Vous développerez un client Java très simple. Il sera composé d’une fenêtre contenant un unique bouton. Lorsque l’on appuie une fois sur ce bouton, l’enregistrement de ce qui est capté par le micro de l’ordinateur débute. Lorsque l’on appuie une seconde fois sur le bouton, l’enregistrement s’arrête et le signal est envoyé à la Servlet centrale pour traduction. Le signal reçu en retour est alors lu sur les enceintes de l’ordinateur. 

VIII. Phase de développement 
Afin de simplifier la phase de développement, vous allez utiliser des version spéciales des classes fournies, qui simulent le fonctionnement des différents services. Les valeurs simulées seront très caractéristiques, vous permettant ainsi de debugger facilement votre code. 

La classe Toolbox : 
public static short[] recordMic(int length) public static String playSignal(short[]) 
Ces deux méthodes sont utilisées par le client et permettent respectivement de simuler l’enregistrement du signal capté par le micro de l’ordinateur et de simuler la lecture du signal sur les enceintes. 
La version de développement de la méthode recordMic produira un tableau de short de taille length et dont les valeurs sont croissantes (1,2,3,4,…). 
La version de développement de la méthode playSignal retournera une chaîne indiquant si le signal est fourni en paramètre est bien la version traduite de ce qui a été enregistré avec recordMic. 

La classe SpeerProxy : 
public static String decode(short[] audio) 
La version de développement de cette méthode renverra une String contenant la représentation sous forme de chaîne de caractère des valeurs du tableau de short passé en paramètre. Par exemple, si on lui passe le tableau de shorts issu de Toolbox.recordMic(5) (ie : [0,1,2,3,4]), elle produira la chaine « 0,1,2,3,4 ». 

La classe TradProxy : 
public static String translateFrToEn(String phrase) 
La version de développement de cette méthode s’attend à avoir en entrée une chaine de la forme de celles produites par SpeerProxy.decode(). Si tel est le cas, elle inverse la chaine (par exemple, si on lui passe « 0,1,2,3,4 », elle retourne « 4,3,2,1,0 »). 

La classe TTSProxy : 
public static short[] synthesizeEn(String sentence) 
La version de développement de cette méthode s’attend à avoir en entrée une chaine de la forme de celles produites par TradProxy.translateFrToEn(). Si tel est le cas, elle retourne un tableau d’entiers inversés (ex : [4,3,2,1,0]). 

IX. Généralités 
Cette application met en œuvre les technologies que vous avez vu au cours des différentes UCE de cette UE, vous devez donc vous servir abondamment des cours et TPs correspondants. N’hésitez pas non plus à chercher sur internet la réponse à des problèmes techniques de développement (ex : comment transformer un tableau de shorts en chaine et vice versa). En effet, cette source d’information est très importante pour le développeur et vous devez apprendre à l’utiliser. 

X. Spécificités de CORBA Java 
Le fonctionnement et la démarche de CORBA sont les mêmes quelque soit leur implémentation. Pour créer un service CORBA en Java, vous devrez, comme en C++, suivre les étapes suivantes : 
- Créer l’interface IDL du service 
- Générer automatiquement la structure du code à partir de l’interface. Utilisez ici la commande idlj –fAll <fichier.idl> - Implémenter les méthodes métiers du serveur. Il faut donc écrire une classe XXXImpl qui hérite de la classe abstraite XXXPOA, où XXX est le nom de votre interface IDL, et implémente les méthodes métier. - Implémenter le lanceur qui va instancier le service et le publier sur l’ORB. Il s’agit d’une application Java dont le corps de main ressemble à ceci :  
// créer et initialiser l’ORB avec le service de nommage tnameserv // port du service de nommage String args[] = {"-ORBInitialPort", "9000"}; Properties props = new Properties(); // IP du serveur de nommage props.put("org.omg.CORBA.ORBInitialHost", "127.0.0.1"); ORB orb = ORB.init(args, props);  
// récupérer une reference au root poa et activer le POAManager POA rootpoa = POAHelper.narrow(orb.resolve_initial_references("RootPOA")); rootpoa.the_POAManager().activate();  
// créer l’instance effectuant le travail métier XXXImpl xxx = new XXXImpl();  
// Récupérer la reference vers l’objet org.omg.CORBA.Object ref = rootpoa.servant_to_reference(xxx); XXX href = XXXHelper.narrow(ref);  
// Récupérer une reference vers le service de nommage org.omg.CORBA.Object nameServiceRef =  orb.resolve_initial_references("NameService"); NamingContextExt ncRef = NamingContextExtHelper.narrow(nameServiceRef);  
// publier l’objet dans le service de nommage sous le nom “XXX” String name = "XXX"; NameComponent path[] = ncRef.to_name(name); ncRef.rebind(path, href);  
System.out.println("XXX prêt a être utilisé..");  
orb.run();  
- Avant de lancer le serveur, lancer le service de nommage dans un terminal (tnameserv ORBInitialPort 9000) - Implémenter le client dont le corps ressemble à :  
// créer et initialiser l’ORB avec le service de nommage tnameserv // port du service de nommage String args[] = {"-ORBInitialPort", "9000"}; Properties props = new Properties(); 
// IP du serveur de nommage props.put("org.omg.CORBA.ORBInitialHost", "127.0.0.1"); ORB orb = ORB.init(args, props);  
// Récupère la référence du NameService System.out.println("Connexion au NameService..."); org.omg.CORBA.Object nameServiceRef = orb.resolve_initial_references("NameService") NamingContextExt nc = NamingContextExtHelper.narrow(nameServiceRef);  
// Récupère la référence du service String servName = "XXX"; XXX xxx = XXXHelper.narrow(nc.resolve_str(servName));  
// utiliser le service xxx.truc();  

XI. Conseils 
Je vous conseille de commencer par créer le squelette de la Servlet centrale qui fera le lien entre les différentes applications. Pour la phase de développement vous utiliserez la méthode doGet, qui a l’avantage de pouvoir être appelée directement depuis un navigateur Web. Vous vous servirez de cette Servlet pour tester au fur et à mesure les différents services que vous développerez. 
Une fois que votre Servlet centrale est en place, déclarez dans sa méthode doGet un tableau de shorts fictif (ex : [0,1,2,3,4,5]), qui simule celui que le client aurait du envoyer. Vous pouvez alors commencer à développer les différents services que la Servlet doit utiliser, dans l’ordre logique : d’abord le service de RAP, puis le service de traduction et enfin le service de TTS. 
Quand tous les services annexes sont fonctionnels, vous pouvez déplacer le code de votre méthode doGet dans la méthode doPost et débuter le développement du client Java.

Architectures Distribuées  Partie 2 UCE Applicative

I. Objectifs 
Maintenant que vous avez construit une architecture distribuée hétérogène, l’objectif de cette seconde partie va être de la modifier pour faire face à de nouvelles contraintes. 

II. Transformer le service de synthèse vocale en WebService 
Dans la première partie de l’application vous avez créé un service de synthèse vocale à l’aide de la technologie Servlet. Vous avez pu remarquer que ce choix n’est pas très heureux, notamment parce que le mécanisme d’échange d’informations entre la Servlets et son client ne sont pas pratiques pour transporter des données binaires comme un signal de parole. 
Vous allez donc transformer le Servlet de synthèse vocale que vous avez développé lors de la première partie en un WebService. 

III. Répartition de charge sur plusieurs services TradProxy 
Dans la première partie de l’application vous avez réalisé un service de traduction, TradService, sous la forme d’un EJB. Ce service encapsule la classe TradProxy, qui vous a été fournie, et qui s’occupe de la partie métier du TradService. 
Le processus de traduction est lourd et nécessite donc un temps d’exécution relativement long. Afin d’accélérer ce processus, on souhaite distribuer la charge de traduction sur plusieurs serveurs. Un moyen simple de réaliser cela est d’instancier plusieurs services de traduction (TradService) sur différent serveurs et de router les demandes de traduction vers les serveurs libres. 
Vous développerez donc un TradServiceHub, faisant office d’intermédiaire entre le Servlet central et plusieurs instances de votre service de traduction (TradService), afin de répartir la charge de manière intelligente. TradServiceHub aura la même interface que le TradService de la première partie de l’application afin que le servlet central puisse l’utiliser comme s’il s’agissait d’un TradService normal. 

Vos TradService doivent posséder une méthode distante pour les interroger sur leur disponibilité. De cette manière, votre TradServiceHub pourra, par exemple, interroger les différents TradService et envoyer la demande de traduction au premier qui est disponible. 
De plus, dorénavant, si un TradService est entrain de synthétiser un signal, toute demande concurrente de synthèse résultera en une exception du côté du client (qui est ici TradServiceHub). Concrètement, vous devez modifier l’interface de TradService pour que la méthode de traduction puisse lever une exception (dérivée de RemoteException) dans le cas ou le TradService est déjà occupé à synthétiser un signal. Le schéma ci-dessous reprend de manière synthétique le rôle de TradServiceHub. 
IV. Intégrer la brique logicielle FreeTTS à la place de TTSProxy 

Votre service de synthèse vocale (TTSService) utilise actuellement le composant TTSProxy qui vous a été fourni. Vous allez maintenant remplacer TTSProxy par un vrai synthétiseur vocal : FreeTTS. 
Concrètement, dans votre TTSService, au lieu d’appeler une méthode de TTSProxy pour effectuer la traduction, vous allez directement utiliser le composant FreeTTS fourni. Dans l’archive de FreeTTS vous trouverez un exemple d’utilisation de FreeTTS dans le fichier test.java. 
Avant de pouvoir utiliser FreeTTS, vous devez le compiler. Pour cela exécuter, en ligne de commande,  la commande du Makefile fourni dans l’archive : 
make freetts 

Le rôle de cette commande est de compiler tous les fichiers java se trouvant dans les dossiers com et de. Si vous travaillez avec un IDE, il suffit d’importer les dossiers com et de à la racine de votre projet TTSService et d’ajouter l’archive jsapi.jar au classpath du projet.  
V. Tester dans des conditions réelles 

Afin de tester l’intégralité de votre architecture distribuée hétérogène en conditions réelles, vous pouvez remplacez les classes de développement Toolbox, SpeerProxy et TradProxy par les nouvelles classes fournies. Une fois ces classes remplacées et vos services recompilés et relancés, branchez un micro-casque sur votre ordinateur et dites n’importe quelle phrase en français, elle devrait vous être automatiquement traduite oralement en anglais. 
