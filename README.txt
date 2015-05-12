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
