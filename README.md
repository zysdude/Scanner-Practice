# Görüntü İşleme Ders Uygulaması: Basit Belge Tarayıcı

Bu repo, **Eskişehir Teknik Üniversitesi (ESTÜ) Robotik ve Yapay Zeka** bölümünde aldığım Görüntü İşleme dersindeki teorik bilgileri gerçek bir senaryoda test etmek için hazırladığım küçük bir projedir. 

Haftalarca işlediğimiz farklı konuların (renk uzayları, filtreleme, perspektif) bir araya gelince nasıl bir ürüne dönüştüğünü görmek amacıyla bu "Scanner" simülasyonunu kurguladım.

Neyi Test Ettim? (Ders Konuları)
Laboratuvar föylerinde parça parça uyguladığımız şu teknikleri bir boru hattı (pipeline) haline getirdim:

 (Görüntü Temelleri):** Ham görüntüyü okuma ve işleme kolaylığı için Gri (Grayscale) tona çevirme.
 (Histogram & Eşikleme):** Yazıların kağıt üzerinde daha net seçilmesi için **Adaptive Thresholding** (Adaptif Eşikleme) ile gölge temizleme.
 (Geometrik Dönüşümler):** Yamuk çekilmiş belgeleri tam üstten bakıyormuş gibi hizalamak için **Warp Perspective** (Perspektif Dönüşümü).
 (Morfoloji):** Kenar tespiti (Canny) ve kağıt sınırlarını netleştirmek için morfolojik operatörler.

 Nasıl Çalışıyor?
Kod oldukça basit bir mantıkla çalışıyor:
1. `girdi_fotograflari` klasöründeki görüntüleri tarıyor.
2. Kağıdın 4 köşesini bulup görüntüyü düzeltiyor.
3. Sonucu bir PDF taraması gibi siyah-beyaz yapıp `Taranan_Belgeler` klasörüne kaydediyor.

📝 Notlar
Bu proje tamamen öğrenme ve pratik yapma amaçlıdır. Ders notlarımdaki algoritmaların gerçek hayattaki karşılığını görmek için hazırlanmıştır. :)


