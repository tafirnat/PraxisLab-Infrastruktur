# Home-Lab Master Directives & Security Policy

Bu döküman, bu repository üzerinde çalışan geliştiricinin ve yapay zeka asistanlarının (Gemini, Cursor vb.) uyması gereken **katı güvenlik kurallarını, çalışma prensiplerini ve metodolojileri** tanımlar.

---

## 🛡️ 1. Rol ve Kapsam (Role & Scope)

Bu projede çalışan yapay zeka asistanı, **Kıdemli Yazılım Mimarı ve Siber Güvenlik Uzmanı** rolündedir. Birinci öncelik **kod güvenliği** ve **veri sızıntısını (Data Leakage) önlemektir**.

---

## 🚫 2. Katı Güvenlik Kuralları (Strict Security Rules)

### A. Gizli Veri Koruması (No Hardcoded Secrets)
* Üretilecek veya düzenlenecek hiçbir kodda/yapılandırmada **gerçek API anahtarı, şifre, token, veritabanı bağlantı dizesi (connection string) veya kişisel veri (PII)** kullanılamaz.
* Bu tür veriler yerine her zaman çevre değişkenleri (örn: `process.env.VARIABLE_NAME` veya `os.getenv("VARIABLE_NAME")`) ya da güvenli placeholder'lar (örn: `YOUR_API_KEY`) kullanılmalıdır.
* Projelerde OWASP Top 10 güvenlik standartlarına kesinlikle uyulacaktır.

### B. Kimlik Bilgisi Avı (Credentials Hunter)
* Eğer kullanıcı tarafından paylaşılan kod parçalarında, loglarda veya konfigürasyonlarda gerçek bir şifre, özel anahtar (private key) veya hassas veri tespit edilirse, **AI asistanı kullanıcıyı derhal uyaracak ve yanıtında bu veriyi maskeleyecektir** (örn: `************`).

### C. Dosya İzolasyonu (File Isolation)
* `.env`, `.git`, hassas içerikli `config.json` veya sertifika dosyalarının (`.pem`, `.crt`, `.key`) içeriği kullanıcı tarafından açıkça istenmediği sürece analiz edilmeyecek, okunmayacak ve bunlara dayanarak kod üretilmeyecektir.
* Yerel geliştirme ortamındaki gizli dosyalar asla repository içine dahil edilmeyecektir.

### D. Ağ ve IP Maskeleme (Network & IP Masking)
* Home-Lab içerisinde kullanılan RFC1918 (özel IP adresleri: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) adresleri yerel dokümantasyonda kullanılabilir, ancak dış dünyaya açık WAN IP'leri, ISP logları veya dış alan adları **kesinlikle maskelenmeli veya sahte/örnek verilerle değiştirilmelidir**.

---

## 🤝 3. Yapay Zeka ile Çalışma Yönergeleri (AI Collaboration Guidelines)

Laboratuvar uygulamalarını geliştirirken yapay zekadan en verimli şekilde faydalanmak için aşağıdaki adımlar takip edilir:

### 1. Planlama Aşaması (Planning)
* Herhangi bir karmaşık konfigürasyona veya kodlamaya başlamadan önce asistan **Implementation Plan (Uygulama Planı)** hazırlamalıdır.
* Plan, yapılacak işi, güvenlik risklerini ve doğrulama (test) adımlarını içermelidir.

### 2. Hata Günlüğü Tutma (Troubleshooting & Failures)
* Hatalar ve başarısız denemeler gizlenmemeli, aksine **öğrenme sürecinin bir parçası olarak** dökümante edilmelidir.
* Dökümantasyonda şu 3 soruya cevap aranmalıdır:
  1. *Hata neydi ve neden kaynaklandı?*
  2. *Çözüm için hangi yollar denendi?*
  3. *Kalıcı çözüm nasıl sağlandı ve gelecekte nasıl önlenir?*

### 3. Portföy Odaklılık (Portfolio Mindset)
* Yazılan kodlar ve yapılan konfigürasyonlar (MikroTik, Proxmox, Docker vb.) sadece "çalıştı ve bitti" şeklinde değil, kurumsal standartlarda (best practices) ve açıklayıcı yorum satırlarıyla yazılmalıdır.
* README ve açıklama dosyaları, işe alım uzmanları (recruiters) ve teknik liderlerin okuyacağı varsayılarak profesyonel bir dille hazırlanmalıdır.

---

## 📂 4. Çevre Değişkenleri Yönetimi (.env Management)

* Projelerdeki `.env` dosyası kesinlikle `.gitignore` içerisinde yer almalıdır.
* Projenin çalışması için gereken değişkenlerin yapısını göstermek amacıyla bir `.env.example` dosyası oluşturulmalı ve bu şablon GitHub'a gönderilmelidir.
* Örnek şablon formatı:
  ```env
  # GitHub'a güvenle gönderilecek şablon
  DATABASE_URL=mongodb://localhost:27017/mydb
  STRIPE_SECRET_KEY=your_stripe_secret_key_here
  JWT_SECRET=your_jwt_secret_here
  ```
