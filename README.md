# webserver
Docker Swarm + Nginx + Portainer + Play With Docker


Bu projede amaç docker swarm üzerinde çalışan yedekli web sunucusu hizmeti vermek ve servisleri portainer adlı tool ile yönetebilmektir.

İşe ilk olarak dilediğimiz sayıda Docker Swarm manager ve worker barındıran bir yapı kurmakla başlayabiliriz. Herhangi bir  server üzerinde 

	docker swarm init --advertise adress <adress>
	
komutu kullanarak o instance'ı doğrudan manager node yapabiliriz. Bu instance bize diğer nodeları nasıl worker olarak atayacağımıza dair token'ı sunacaktır. Doğrudan Play-With-Docker ortamını kullanarak 1 manager 1 Worker yapısını otomatik olarak hazırlayıp manager nodda işlemlere başlamak demo çalışması esnasında daha kolay olacaktır, o durumda üstteki gibi yapı kurmadan yolumuza devam edebiliriz.

Yapıyı kurduktan sonra herhangi bir manager node üzerinde portainer'ı kurmamız gerekecek. Kurulacak olan portainer her noda agent atayarak ilgili nodu yönetmememizi sağlayacaktır. Portainer'ı kurmak için öncelikle yukarıdaki portainer-agent-stack.yml dosyasını çalıştıracağımız foldera ekleyeceğiz. Ardından aşağıdaki komutu kullanarak portainer'ı ayaklandırıyoruz.

	docker stack deploy -c portainer-agent-stack.yml portainer

Referans: https://docs.portainer.io/v/ce-2.9/start/install/server/swarm/linux

Artık portainer tüm nodlara erişir vaziyette hazır bir şekilde bizi bekliyor. Portainera ulaşmak için localhost:9000 üzerinden giriş yapmamız yeterli olacaktır.

Bu noktada swarm yapımızı kurduk ve izleyebiliyoruz. Şimdi sırada sunacağımız web sayfasının nginx üzerinden servis olarak sunulması işlemi var. Docker swarm üzerinden kopyalı(replicas) bir şekilde servis sunmak için aşağıdaki komutu kullanabiliriz. 

	docker service create --publish mode=host,target=80,published=8080 --replicas=1 --name myservice nginx

Bu komuttan da anlaşılacağı üzere;
swarm yapısına bir servis oluşturması gerektiğini,
bu servisin 8080 üzerinden gelen istekleri container üzerinden 80 portuna yönlendirmesi gerektiğini,
Aynı servisten 3 kopya oluşturması gerektiğini (bunu manager  dağıtacak diğer nodelara),
Servis isminin myservice olması gerektiği,
Ve son olarak nginx imageının kullanılması gerektiğini belirtmiş olduk.

Her şey doğru ise localhost:8080 üzerinden default nginx sayfasını görebiliriz.

Şimdi sıra geldi istediğimiz bir sayfayı nginx üzerine kopyalamaya. Bu adımda ister manager node üzerinden ister portainer üzerinden nginx cli açmak gerekiyor. Manager node üzerinden açmak için aşağodaki komutlar kullanılabilir: 
	
	docker ps
	docker exec -it <container_id> bin/bash/

Portainer portalından ise Container sekmesinden açabiliriz. İlgili containera gelip cli ikonuna tıklayarak containerın içine girebiliriz.

Girdikten sonra index dosyasının olduğu bölüme gidip oradaki dosyalar ile kendi dosyalarımız değiştirmemiz gerekecek. Nginx default olarak html dosyalarını cd usr/share/nginx/html/  adresinde tutar.  Bu komutla ilgili adrese gidip mevcut index i silebiliriz.

Şimdi son olarak webde hazır bulunan bir statik site dosyalarını  bu lokasyona çekme işlemine bakalım. Aşağıdaki komutlaru sırasıyla çalıştıralım.

	apt-get update
	apt-get install vim
	apt-get install wget
	apt-get install unzip 

Bu komutlar sayesinde artık wget kullanarak istenen url den buraya file download edebiliriz. İndirilen zip dosyalarını açabiliriz. Örnek olarak aşağıdaki siteyi download edelim

	wget https://github.com/startbootstrap/startbootstrap-creative/archive/gh-pages.zip

	unzip gh-pages.zip
	rm gh-pages.zip
	mv start-bootstrap-template/* . #hedef içindeki tüm dosyaları buraya getir.

Artık site hazır portainer üzerinden görsel olarak view edip istediğimiz servisi kapatıp açıp olanlara tanıklık edebiliriz.






