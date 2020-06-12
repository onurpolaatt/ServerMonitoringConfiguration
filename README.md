# ServerMonitoringConfiguration
Bu dokümanda; Server makinelerinin tüm bilgilerini monitör etmek, çalışma şekli ve konfigürasyonları anlatılacaktır.
# ÖZELLİKLER
Prometheus, Alertmanager ve Grafana bir arada çalışan sistemler bütünüdür.

**Grafana;**  alınan tüm bilgileri görselleştirilir. İstenilen şekle sokarak, takip edilmesini ve okunmasını daha kolay hale getirir.
![GRAFANA1](https://github.com/onurpolaatt/ServerMonitoringConfiguration/blob/master/pictures/1.png)

Tablo şeklinde gördüğümüz tüm bilgiler, **node_exporter** tarafından elde ettiğimiz bilgilerdir. Node_exporter kurulu olduğu sistemi dinleyerek tüm bilgileri dışarıya paylaşan bir sistemdir. Prometheus ile doğrudan çalışmaktadır. Bir port ile bu bilgileri dışarıya aktarır.
---
**Prometheus;** Hangi portları dinlememiz gerektiğini, dinlediğimiz bu portlarda çalışan entegre sistemlerle (node_exporter vb.) neler yapabileceğimizi söyleyen, herhangi bir hata durumunda tanımladığımız kural setlerini devreye sokan ve alertmanager ile konuşan monitoring toolumuzun ana yapısıdır.

![PROMETHEUS1](https://github.com/onurpolaatt/ServerMonitoringConfiguration/blob/master/pictures/2.png)
