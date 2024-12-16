### Kubernetes Üzerinde Pega Platform Sıfır Kesinti ile Güncelleme Süreci (Türkçe Özet)

#### Genel Bakış
Pega Platform™, Kubernetes ortamınızda sıfır kesinti ile (zero-downtime) güncellenebilir. Bu süreç, Pega'nın Docker imajlarına dayalı olarak Helm Chart'ları kullanarak otomatikleştirilmiştir. Güncelleme işlemi sırasında sistem kullanılabilir durumda kalır ve uygulamada çalışmaya devam edebilirsiniz. Bu yöntem, **Pega Infinity™ 8.4.2** ve üzeri sürümler için geçerlidir.

#### Güncelleme Süreci
Güncelleme işlemi aşağıdaki dört şemayı kullanarak gerçekleştirilir:
1. **Data Schema:** Mevcut veri şeması, güncellemeden sonra aynı kalır.
2. **Rules Schema:** Mevcut kurallar şeması, güncelleme sırasında yükseltilir.
3. **Geçici Veri Şeması (Dtmp):** Geçici tablo verileri için kullanılan, yükseltme sonrası silinmesi gereken bir şema.
4. **Yeni Kurallar Şeması (Rn):** Kuralların yedeklendiği ve güncellemeden sonra kalıcı hale gelen yeni şema.

#### Güncelleme Adımları
1. **Hazırlık:**
   - Yeni kurallar ve geçici veri şemalarını veritabanınızda oluşturun.
   - Docker deposunu en son Pega Docker imajlarını indirerek hazırlayın.
2. **Helm Ayarlarının Güncellenmesi:**
   - `pega.yaml` dosyasındaki ayarları düzenleyin.
   - Aşağıdaki parametreleri özelleştirin:
     - **actions.execute:** `upgrade-deploy`
     - **jdbc.rulesSchema:** Mevcut kurallar şemanız.
     - **jdbc.dataSchema:** Mevcut veri şemanız.
     - **installer.upgradeType:** `zero-downtime`
     - **installer.targetRulesSchema:** Yeni kurallar şema adı.
     - **installer.targetDataSchema:** Yeni veri şema adı.
3. **Güncellemenin Başlatılması:**
   - `helm upgrade` komutuyla yükseltmeyi başlatın.
   - Kubernetes Dashboard üzerinden güncelleme durumunu izleyin.
4. **Son Kontroller ve Temizlik:**
   - Güncelleme tamamlandıktan sonra geçici veri şemasını silin.
   - Güncelleme sırasında sistem yük dengeleme ve erişilebilirlik özelliklerini otomatik olarak düzenler.

#### Ek Bilgiler
- **Gereksinimler:** Pega Platform 8.4.2 veya üzeri bir sürüm, Helm Chart 1.6.0 veya daha yenisi.
- **Kafka Desteği:** 8.8 ve üzeri sürümler için Kafka dışa aktarılmış bir yapılandırma gereklidir.
- **Süre:** Süreç toplamda yaklaşık 3 saat sürer, ancak sistem yapılandırmanıza bağlı olarak değişebilir.

#### Hata Durumunda Devam
- Hata durumunda, **rules_upgrade** ve **data_upgrade** adımları ayrı ayrı çalıştırılabilir.
- Güncelleme kaldığı yerden devam eder; tamamlanan adımlar atlanır.

#### Önemli Komutlar
- Helm deposunu güncellemek için:
  ```bash
  helm repo update pega https://pegasystems.github.io/pega-helm-charts
  ```
- Güncellemeyi başlatmak için:
  ```bash
  helm upgrade mypega-<platform>-demo pega/pega --namespace mypega-<platform>-demo --values pega.yaml
  ```
