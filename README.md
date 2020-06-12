# ServerMonitoringConfiguration
Bu dokümanda; Server makinelerinin tüm bilgilerini monitör etmek, çalışma şekli ve konfigürasyonları anlatılacaktır.
# ÖZELLİKLER
Prometheus, Alertmanager ve Grafana bir arada çalışan sistemler bütünüdür.

**Grafana;**  alınan tüm bilgileri görselleştirilir. İstenilen şekle sokarak, takip edilmesini ve okunmasını daha kolay hale getirir.

![GRAFANA1](https://github.com/onurpolaatt/ServerMonitoringConfiguration/blob/master/pictures/1.png)

Tablo şeklinde gördüğümüz tüm bilgiler, node_exporter tarafından elde ettiğimiz bilgilerdir. Node_exporter kurulu olduğu sistemi dinleyerek tüm bilgileri dışarıya paylaşan bir sistemdir. Prometheus ile doğrudan çalışmaktadır. Bir port ile bu bilgileri dışarıya aktarır.

---

**Prometheus;** Hangi portları dinlememiz gerektiğini, dinlediğimiz bu portlarda çalışan entegre sistemlerle (node_exporter vb.) neler yapabileceğimizi söyleyen, herhangi bir hata durumunda tanımladığımız kural setlerini devreye sokan ve alertmanager ile konuşan monitoring toolumuzun ana yapısıdır.

![PROMETHEUS1](https://github.com/onurpolaatt/ServerMonitoringConfiguration/blob/master/pictures/2.png)

Belirtilen hedefler doğrultusunda neleri yapıp yapamayacağımıza ulabiliriz. Örnek; 

![PROMETHEUS2](https://github.com/onurpolaatt/ServerMonitoringConfiguration/blob/master/pictures/3.png)

Yukarıdaki örnekte görüldüğü üzere; config dosyasında belirttiğimiz hedeflerimizi, prometheusun “up” sorgusunu kullanarak görebiliyoruz. Burada belirtilen sorguyu da **Grafana** üzerinden tablo şeklinde görebiliriz.

![GRAFANA2](https://github.com/onurpolaatt/ServerMonitoringConfiguration/blob/master/pictures/4.png)

Prometheus üzerindeki  **“Alerts”** alanında ise; Config dosyamızda belirttiğimiz, kural setlerimiz mevcuttur. Bu kurallar istenilen şekilde modifiye ve gruplandırma edilmekle birlikte, paralel olarak **alertmanager** toolumuzla iletişim kurup, kural devreye girdiğinde ne yapması gerekiyorsa gerçekleştirmesini sağlamaktadır.

![PROMETHEUS3](https://github.com/onurpolaatt/ServerMonitoringConfiguration/blob/master/pictures/5.png)

Yukarıdaki görselde, prometheus üzerinede tanımlanan target’ların herhangi birinde cevap alamama durumunu kontrol eden bir kural tanımlanmıştır. Eğer kuralda belirtildiği gibi bir target’tan 1 dakika cevap alamadığı durumda alertmanager toolumuzla iletişime geçmeye çalışacaktır.

----

**Alertmanager;** Herhangi bir alert tetiklendiğinde ne yapılması gerektiğini belirttiğimiz toolumuzdur. Prometheus üzerinden alertmanager’a bir başvuru geldiğinde, konfigürasyon dosyasında belirttiğimiz ayarlar devreye girerek ilgili işlemi gerçekleştirilir.

![ALERTMANAGER1](https://github.com/onurpolaatt/ServerMonitoringConfiguration/blob/master/pictures/6.png)

Yukarıdaki görselde, alertmanager tetiklendiğinde yapması gereken konfigürasyonlar görülmektedir. Bizim tanımladığımız ‘instanceDown’ kuralı devreye girdiğinde slack kanalına bildirim göndermesi istenmiştir. Slack kanalına bildirim gönderilirken, kanala basılacak mesaj biçimi, mesaj içerikleri vb. gibi konfigürasyonlar da tanımlanmaktadır.

![SLACK1](https://github.com/onurpolaatt/ServerMonitoringConfiguration/blob/master/pictures/7.png)

# KURULUM
Prometheus, Alertmanager ve Node_exporter macOS tabanlı işletim sistemlerinde sadece docker üzerinde çalıştığı için docker-compose yapısı kullanılarak kurulumlar gerçekleştirilmiştir.

* **NODE_EXPORTER:**

Node-exporter ‘darwin' ve ‘linux’ tabalı işletim sistemlerinde çalışmaktadır. Bizim server’ımız linux işletim sistemine sahiptir.

- Servera bağlandıktan sonra ana dizine node_exporter isimli bir klasör oluşturup içerisine giriyoruz. (“mkdir node_exporter” – “cd /node-exporter”)
- [Prometheus’un](https://prometheus.io/download/) sitesinde belirtilen node_exporter'ın linux için indirme linki kopyalanıp, server içerisine kurulur. ( “wget  https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz ”)
- Yükleme bittikten sonra “tar -zxvf node_exporter-0.18.1.linux-amd64.tar.g“ komutu çalıştırılır.
- “cd node_exporter-0.18.1.linux-amd64/” komutu ile oluşan klasörün içerisine girilir.
- “ ./node_exporter &” komutu ile node_exporter çalışmaya başlatılıp hangi portu dinlediği belirlenir. Port belirlendikten sonra dışarıya açmak için izin verilir.

![SERVER1](https://github.com/onurpolaatt/ServerMonitoringConfiguration/blob/master/pictures/8.png)

“ufw allow 9100” komutu çalıştırılarak firewall izni verilir.  
Tüm bu işlemler sonrasında node_exporter 9100 portunda çalışmaya devam eder. (Sağlaması için ‘netstat -pltn’ komutu ile görebilirsiniz.)

![SERVER2](https://github.com/onurpolaatt/ServerMonitoringConfiguration/blob/master/pictures/9.png)

NOT: Eğer node_exporter çalışmayı durdurursa, server makinamızda oluşturduğumuz klasörün içine girip ( cd node_exporter/ ----  cd node_exporter-0.18.1.linux-amd64/) “./node_exporter &”  komutunu çalıştırarak tekrar ayağa kaldırabilirsiniz.

* **PROMETHEUS - ALERTMANAGER VE GRAFANA:**

Bu kurulumları gerçekleştirmek için docker-compose yapısı kullanıldı;
Kullanılan Yapının bir örneği aşağıda eklidir;

```
version: '3.1'

volumes:
    prometheus_data: {}
    grafana_data: {}

services:

  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus/:/etc/prometheus/
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    ports:
      - 9090:9090
    links:
      - alertmanager:alertmanager
    restart: always
    deploy:
      mode: global


  alertmanager:
    image: prom/alertmanager:latest
    ports:
      - 9093:9093
    volumes:
      - ./alertmanager/:/etc/alertmanager/
    restart: always
    command:
      - '--config.file=/etc/alertmanager/config.yml'
      - '--storage.path=/alertmanager'
    deploy:
      mode: global

   grafana:
     image: grafana/grafana:5.4.2
     depends_on:
       - prometheus
     ports:
       - 3000:3000
     volumes:
       - grafana_data:/var/lib/grafana
       - ./grafana/dashboards:/var/lib/grafana/dashboards
       - ./grafana/provisioning:/etc/grafana/provisioning

```

Compose dosyamızın çalıştığı klasörün içerisinde iki farklı klasör daha bulunmaktadır. Birincisi **prometheus** isimli klasör ve içerisinde  prometheus.yml ve rules.yml isimli dosyalar barındırır. Kural tanımlarını ve prometheusun hedeflerini konfigüre ettiğimiz dosyalarımızdır.
- prometheus.yml
```
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    monitor: 'codelab-monitor'
    
rule_files:
   - rules.yml

alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - alertmanager:9093

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets:
        - localhost:9090
  - job_name: 'node_exporter'
    scrape_interval: 5s
    static_configs:
      - targets:
        - SERVERIP:9100
  - job_name: 'alertmanager'
    scrape_interval: 5s
    static_configs:
      - targets:
        - alertmanager:9093
```
- rules.yml

```
groups:
 - name: instanceDownAlert
   rules:
   - alert: InstanceDown
     expr: up == 0
     for: 1m
     labels:
       serverity: "critical"
     annotations:
       summary: "Endpoint {{ $labels.instance }} down"
       description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minutes."
```
Diğer klasörümüzün ismi ise **alertmanager**. Bu klasörün içerisinde de config.yml isimli dosyamız mevcuttur. İçerisinde alert tetiklendiği durumda ne aksiyon alması gerektiğinin konfigürasyonu bulunmaktadır.

```
global:
  slack_api_url: 'https://hooks.slack.com/services/*****'
route:
  group_by: ['instance', 'severity']
  group_wait: 30s
  group_interval: 30s
  repeat_interval: 30s
  routes:
  - match:
      alertname: InstanceDown
  receiver: 'alert-team'
receivers:
- name: 'alert-team'
  slack_configs:
  - channel: '#builds'
    icon_url: https://avatars3.githubusercontent.com/u/3380462
    title: |-
     [{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}] {{ .CommonLabels.alertname }} for {{ .CommonLabels.job }}
     {{- if gt (len .CommonLabels) (len .GroupLabels) -}}
       {{" "}}(
       {{- with .CommonLabels.Remove .GroupLabels.Names }}
         {{- range $index, $label := .SortedPairs -}}
           {{ if $index }}, {{ end }}
           {{- $label.Name }}="{{ $label.Value -}}"
         {{- end }}
       {{- end -}}
       )
     {{- end }}
    text: >-
     {{ range .Alerts -}}
     *Alert:* {{ .Annotations.title }}{{ if .Labels.severity }} - `{{ .Labels.severity }}`{{ end }}

     *Description:* {{ .Annotations.description }}

     *Details:*
       {{ range .Labels.SortedPairs }} • *{{ .Name }}:* `{{ .Value }}`
       {{ end }}
     {{ end }}
```
**NOT:**  “prometheus.yml” isimli dosyamızda alerting kısmında belirtilen target alanının diğerlerinden farklı olmasının sebebi; Farklı docker containerlar içerisinde bulundukları için, birbirleri ile konuşmalarını sağlamak ve ilgili portu düzgün biçimde belirtmektir. ( diğerleri localhost:**** şeklinde tanımlanmışken alertmanager => alertmanager:9093 şeklinde tanımlanır.)

---

# NODE_EXPORTER SERVİS HALİNE GETİRME:
node_exporter server makinesinde çalışırken, kendiliğinden kapanma durumu yaşanabiliyor. Bunun önüne geçmek için servis haline getirmemiz gerekecektir.
* Yüklemeyi yaptığımız klasörden node_exporter kopyalanır: “cp /node_exporter/node_exporter-0.18.1.linux-amd64/node_exporter /usr/local/bin/node_exporter”
* systemd altına bir servis dosyası oluşturulur: “sudo nano /etc/systemd/system/node-exporter.service”
* Oluşturulan servis dosyasının içerisine ise aşağıdaki bilgiler girilir;

```
[Unit]
Description=Prometheus Node Exporter Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```
* Her şey hazır. Son olarak “sudo service node-exporter start” komutunu çalıştırarak başlatabilirsiniz. “sudo service node-exporter status” komutu ile de durumunu inceleyebilirsiniz.

Hepinize İyi Çalışmalar.

(W)