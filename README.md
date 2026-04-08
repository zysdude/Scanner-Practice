import cv2
import numpy as np
import os
import glob
import matplotlib.pyplot as plt

def order_points(pts):
    """Köşe noktalarını sıralar: sol-üst, sağ-üst, sağ-alt, sol-alt"""
    rect = np.zeros((4, 2), dtype="float32")
    s = pts.sum(axis=1)
    rect[0] = pts[np.argmin(s)]
    rect[2] = pts[np.argmax(s)]
    diff = np.diff(pts, axis=1)
    rect[1] = pts[np.argmin(diff)]
    rect[3] = pts[np.argmax(diff)]
    return rect

def scan_single_document(image_path):
    """Tek bir görüntüyü işler, taranmış halini ve orijinalini döndürür."""
    image = cv2.imread(image_path)
    if image is None:
        return None, None
        
    orig = image.copy()
    gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    blurred = cv2.GaussianBlur(gray, (5, 5), 0)
    edged = cv2.Canny(blurred, 75, 200)

    contours, _ = cv2.findContours(edged.copy(), cv2.RETR_LIST, cv2.CHAIN_APPROX_SIMPLE)
    contours = sorted(contours, key=cv2.contourArea, reverse=True)[:5]

    document_contour = None
    for c in contours:
        peri = cv2.arcLength(c, True)
        approx = cv2.approxPolyDP(c, 0.02 * peri, True)
        if len(approx) == 4:
            document_contour = approx
            break

    if document_contour is None:
        return None, orig # Kağıt bulunamadı

    # A4 formatı boyutları
    hedef_genislik, hedef_yukseklik = 500, 700
    hedef_noktalari = np.array([
        [0, 0],
        [hedef_genislik - 1, 0],
        [hedef_genislik - 1, hedef_yukseklik - 1],
        [0, hedef_yukseklik - 1]
    ], dtype="float32")

    kaynak_noktalari = order_points(document_contour.reshape(4, 2))
    M = cv2.getPerspectiveTransform(kaynak_noktalari, hedef_noktalari)
    warped = cv2.warpPerspective(orig, M, (hedef_genislik, hedef_yukseklik))

    warped_gray = cv2.cvtColor(warped, cv2.COLOR_BGR2GRAY)
    scanned_result = cv2.adaptiveThreshold(warped_gray, 255, 
                                           cv2.ADAPTIVE_THRESH_GAUSSIAN_C, 
                                           cv2.THRESH_BINARY, 21, 10)
    return scanned_result, orig

def batch_process_colab(input_folder="girdi_fotograflari", output_folder="Taranan_Belgeler"):
    """Klasördeki görüntüleri tarar, kaydeder ve ekranda çizer."""
    
    # Klasörler yoksa oluştur
    if not os.path.exists(input_folder):
        os.makedirs(input_folder)
        
    if not os.path.exists(output_folder):
        os.makedirs(output_folder)

    # Desteklenen formatları bul (Büyük/Küçük harf duyarlılığı için eklentiler artırıldı)
    image_files = []
    for ext in ('*.jpg', '*.jpeg', '*.png', '*.JPG', '*.JPEG', '*.PNG'):
        image_files.extend(glob.glob(os.path.join(input_folder, ext)))

    if not image_files:
        print(f"⚠️ '{input_folder}' klasöründe fotoğraf bulunamadı!")
        print("Lütfen sol taraftaki dosya menüsünden klasörü açıp içine test fotoğrafları yükleyin ve kodu tekrar çalıştırın.")
        return

    print(f"🚀 Toplam {len(image_files)} dosya işleniyor...\n" + "-"*40)

    for file_path in image_files:
        dosya_adi = os.path.basename(file_path)
        print(f"⏳ İşleniyor: {dosya_adi}...")
        
        sonuc, orig = scan_single_document(file_path)
        
        if sonuc is not None:
            kayit_yolu = os.path.join(output_folder, f"tarandi_{dosya_adi}")
            cv2.imwrite(kayit_yolu, sonuc)
            
            # --- COLAB EKRANINDA GÖSTERME (Matplotlib) ---
            fig, axes = plt.subplots(1, 2, figsize=(12, 6))
            
            # Orijinal Görüntü (OpenCV BGR okur, Matplotlib RGB ister. Bu yüzden çeviriyoruz)
            axes[0].imshow(cv2.cvtColor(orig, cv2.COLOR_BGR2RGB))
            axes[0].set_title(f"Orijinal: {dosya_adi}")
            axes[0].axis('off')
            
            # Taranmış Görüntü
            axes[1].imshow(sonuc, cmap='gray')
            axes[1].set_title("Taranmış Sonuç (PDF Formatı)")
            axes[1].axis('off')
            
            plt.show()
            print("✅ Başarılı!\n")
        else:
            print("❌ Atlandı (Fotoğrafta 4 köşeli net bir kağıt tespit edilemedi).\n")

# Kodu başlat
batch_process_colab("girdi_fotograflari", "Taranan_Belgeler")


