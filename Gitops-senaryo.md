---
title: Gitops
--- 

# GitOps ve Argo CD
## Selamlar, GitOps prensibi ve ArgoCD aracı hakkında öğrenim senaryosudur. 
###  Seviye: Başlangıç

## Neden GitOps?

![what_is_argo](./gitops.png)

Günümüzde hızla değişen ve genişleyen yazılım geliştirme dünyasında, dinamik bir yapıda olmak ve yeniliklere adapte olmak oldukça önemli. Zaman içerisinde, ihtiyaçları hızlı karşılayamadığımız Waterfall modelinden Agile’a geçiş yaptık. Daha küçük değişikliklerle, küçük iterasyonlarla hedefi yakalamaya başladık. Bu çevik yapının içinde, sürekli değişimleri karşılamak adına operasyon ve geliştirme takımlarının iç içe geçtiği devops kültürünü oluşturduk. Günümüz modern dünyasında ise daha büyük challengelar ile başbaşayız. Containerization’ın yaygınlaşması, Kubernetes’in hayatımıza girmesiyle beraber, herhangi bir Git reposu üzerinde tutulan uygulamaların bu Kubernetes cluster ları üzerine merkezi bir yerden deploy olması ve yönetilmesi bunlardan biri. GitOps’a bu yüzden ihtiyaç duyuyoruz.

## GitOps Nedir?

Genel bir tanımla; GitOps, versiyon kontrol, işbirliği, uyumluluk ve CI/CD araçları gibi uygulama geliştirme için kullanılan DevOps best practice'lerini alan ve bunları altyapı otomasyonuna uygulayan operasyonel bir çerçevedir.

## Argo CD Nedir?

![what_is_argo](./img8.png)

Argo CD; Kubernetes için bildirime dayalı, GitOps metodolojisini izleyen bir sürekli teslim aracıdır.

“Uygulama tanımları, konfigürasyonları ve ortamları bildirime dayalı ve versiyon kontrollü olmalıdır” mantığı ile çalışır. Uygulama dağıtımı ve yaşam döngüsü yönetiminin otomatikleştirilmiş, denetlenebilir ve anlaşılabilir olmasını sağlar.

## Argo CD Nasıl Çalışır?

![how_argo_works](./img7.png)

Argo CD belirli periyotlarda Kubernetes’e deploy edilmiş uygulamaları izler ve Git reposunda yer alan tanım dosyalarıyla karşılaştırmalar yapar. Herhangi bir farklılık durumunu kullanıcıyı bildirir ya da belirtilirse otomatik olarak senkronizasyon işlemini gerçekleştirir.

## Neden Argo CD ?

+ Kubernetes cluster'ınızı görselleştirir. Yani bir deployment inizi replica set,pod vb.kuberetes objeleri şeklinde görselleştiriyor.
+ Helm, customize, yaml tanım dosyalarını destekliyor.
+ Birden fazla kubernetes cluster ile entegre edilebiliyor.
+ İmperative bir şekilde kubectl kullanmıyorsunuz, yapınızı declarative yapıyorsunuz yani yaptığınız değişikliği Argo CD algılayıp Kubernetes’e yüklenmesini sağlıyor.
+ Kubernetes cluster'ınızda bir deployment silindi veya bir kesinti yaşadınız; Argo CD hem git reposu hem Kubernetes'i sürekli monitor ettiğinden ikisi arasında fark olduğunu anlayıp silinen deployment'ı tekrar yerine getirebiliyor.

## Öğrenme Hedefleri

GitOps senaryosunu tamamladıktan sonra;

+ GitOps prensibinin ne olduğu,
+ ArgoCD aracının ne işe yaradığıu ve nasıl kurulduğu hakkında bilgi sahibi olacaksınız.

## Ön gereksinimler

+ Linux dağıtımlarından en az birine aşina olmalı ve kullanabiliyor olmalısınız.
+ Docker ve podman gibi container araçlarını kullanabiliyor olmalısınız.
+ Kubernetes container orkestrasyonu hakkında bilgi sahibi olmalısınız.

## Senaryo 1

+ Argo CD Kurulumu

Uygulamaları dağıtabilmemizi sağlayan Argo CD aracını clusterımıza yükleyelim. Halihazırda Kubernetes kurulu ortamımızda aşağıdaki adımları izleyerek Argo CD kurabiliriz.

Öncelikle argocd isminde bir namespace oluşturalım. `kubectl create namespace argocd`

`kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml` komutu ile Argo CD GitHub sayfasında yer alan tanım dosyaları ile kurulumu gerçekleştirebiliriz.

`kubectl get all -n argocd` komutu ile Argoocd namespace'inde oluşturulan objeleri listeleyelim.

Varsayılan olarak Argo CD API sunucusu harici bir IP ile ile expose edilmez. Bunun için port-forwarding ile API Server'a bağlanacağız.

`kubectl port-forward svc/argocd-server -n argocd 8080:443`

Artık API Server'a https://localhost:8080 üzerinden erişilebilir. Gidip ArgoCD arayüzü ile tanışma zamanı!

Admin hesabının ilk parolası otomatik olarak oluşturulur ve Argo CD kurulum ad alanınızda argocd-initial-admin-secret adlı bir gizli alan parolasında açık metin olarak saklanır. Bu şifreyi kubectl kullanarak kolayca alabilirsiniz. Başka bir terminal tab'ı içerisinde aşağıdaki komutu çalıştırın.

`kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo`

Artık kullanıcı admin olan Argocd hesabının password bilgisine sahipsiniz. Bu bilgiler ile https://localhost:8080 adresinden ArgoCD'ye giriş sağlayabilirsiniz.

Bizleri aşağıdaki gibi bir Argo CD arayüzü karşılıyor olacak!

![argo_homepage](./screenpage.png)

## Senaryo 2

+ Argo CD Üzerinde Uygulama Oluşturma

Merhabalar, Kubernetes ortamı üzerinde Argo CD kurulumumuzu tamamladığımıza göre artık ilk Argo CD uygulamamızı deploy edebiliriz.

Argo CD arayüzü üzerinde bir uygulama oluşturmak için "+New App" butonuna tıklanır. Açılan pencere üzerinden uygulama bilgileri doldurulur. Bu örnekte Codefresh'in örnek reposundaki bir uygulamayı deploy edeceğiz. 

Aşağıda açılan pencere üzerinde uygulama bilgilerini girelim:

![argo_create_app](./img1.png)

![argo_create_app2](./img2.png)

Son durumda Appliation bilgileri aşağıdaki gibidir:

+ application name : `demo`
+ project: `default`
+ repository URL: `https://github.com/codefresh-contrib/gitops-certification-examples`
+ path: `./simple-app`
+ Cluster: `https://kubernetes.default.svc` 
+ namespace: `default`

Diğer parametreleri boş bırakarak Create butonuna tıkladığımızda demo uygulamamız aşağıdaki şekilde oluşur.

![argo_create_app3](./img3.png)

Argo CD bizlere uygulamamızı otomatik veya manuel şekilde sync etme seçeneği sunar. Şanki uygulamamızda sync durumu manuel olacak şekilde bir seçim yaparak ilerledik. Yani biz manuel olarak tetiklemediğimiz sürece uygulama sync durumuna geçmeyecektir.

Uygulama şuanda OutOfSync durumundadır. Bunun anlamı;

+ Cluster boş
+ Git reposunda bir uygulama var
+ Bu nedenle Git durumu ve cluster durumu farklıdır. Git reposu ve cluster sync durumda değildir. (OutOfSync)

Ayrıca CLI’da aşağıdaki komutu çalıştırarak herhangi bir deployment oluşmadığını da görebiliriz.

`kubectl get deployments` 

![argo_create_app4](./img4.png)

Application’ı oluştururken sync policy parametresini manuel olarak seçtiğimiz için, sync butonuna basarak uygulamayı manuel olarak tetiklemeliyiz. Sync olduktan sonra uygulamamız içine girdiğimizde aşağıdaki şekilde gözükecektir.

![argo_create_app5](./img6.png)

Tüm kalper yeşil! Uygulamamız başarıyla deploy oldu!

Tekrar `kubectl get deployments` dediğimizde oluşan uygulamamızı artık görüntüleyebiliriz.

Burada aynı zamanda Argo CD arayüzü sayesinde görselleştirilmiş bir cluster görebiliyoruz. Demo uygulamamızda yer alan,
replicaset, endpoint, pod objelerini görüntüleyebiliyoruz. Uçtan uca görselleştirilmiş bir deployment!

![argo_create_app6](./img5.png)

Arayüz üzerinden “delete” butonuna basarak uygulamayı silip, demo çalışmasını bitirebiliriz. 









