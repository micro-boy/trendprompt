# Adobe Stock Trending Images Scraper

Panduan lengkap untuk melakukan web scraping pada Adobe Stock untuk tujuan pembelajaran.

## ⚠️ Disclaimer

**PENTING**: Script ini dibuat khusus untuk tujuan edukasi dan pembelajaran. Pastikan untuk:
- Mematuhi Terms of Service Adobe Stock
- Menggunakan rate limiting yang wajar
- Tidak melakukan download gambar tanpa lisensi
- Mempertimbangkan penggunaan Adobe Stock API resmi

## 📋 Prerequisites

### Instalasi Python Libraries

```bash
pip install requests beautifulsoup4 pandas lxml
```

### Import Libraries yang Diperlukan

```python
import requests
from bs4 import BeautifulSoup
import time
import json
import pandas as pd
from urllib.parse import urljoin, urlparse
import os
```

## 🔧 Kode Lengkap

### Class Utama: AdobeStockScraper

```python
class AdobeStockScraper:
    def __init__(self):
        self.base_url = "https://stock.adobe.com"
        self.session = requests.Session()
        # Set user agent untuk menghindari blocking
        self.session.headers.update({
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36'
        })
        
    def get_trending_images(self, category="photos", limit=50):
        """
        Mengambil data gambar trending dari Adobe Stock
        
        Args:
            category: jenis konten (photos, vectors, illustrations)
            limit: jumlah maksimal gambar yang diambil
        """
        trending_url = f"{self.base_url}/trending/{category}"
        
        try:
            print(f"Mengakses: {trending_url}")
            response = self.session.get(trending_url, timeout=10)
            response.raise_for_status()
            
            soup = BeautifulSoup(response.content, 'html.parser')
            images_data = []
            
            # Cari container gambar (struktur bisa berubah)
            image_containers = soup.find_all('div', {'data-testid': lambda x: x and 'asset-tile' in x}) or \
                             soup.find_all('div', class_=lambda x: x and 'asset' in x.lower()) or \
                             soup.find_all('img', src=lambda x: x and 'adobe' in x)
            
            print(f"Ditemukan {len(image_containers)} gambar")
            
            for i, container in enumerate(image_containers[:limit]):
                if i % 10 == 0:
                    print(f"Progress: {i}/{min(limit, len(image_containers))}")
                
                image_data = self.extract_image_info(container)
                if image_data:
                    images_data.append(image_data)
                
                # Rate limiting - jangan terlalu cepat
                time.sleep(0.5)
                
            return images_data
            
        except requests.RequestException as e:
            print(f"Error saat mengakses website: {e}")
            return []
    
    def extract_image_info(self, container):
        """Extract informasi dari setiap gambar"""
        try:
            image_info = {}
            
            # Cari tag img
            img_tag = container.find('img') if container.name != 'img' else container
            
            if img_tag:
                # URL thumbnail
                image_info['thumbnail_url'] = img_tag.get('src') or img_tag.get('data-src')
                
                # Alt text sebagai deskripsi
                image_info['description'] = img_tag.get('alt', 'No description')
                
                # Cari link detail
                link_tag = container.find_parent('a') or container.find('a')
                if link_tag:
                    image_info['detail_url'] = urljoin(self.base_url, link_tag.get('href', ''))
                
                # Cari informasi tambahan
                parent = container.find_parent()
                if parent:
                    # Cari download count atau popularity indicator
                    download_info = parent.find(text=lambda x: x and ('download' in x.lower() or 'view' in x.lower()))
                    if download_info:
                        image_info['popularity'] = download_info.strip()
                
                # Extract ID dari URL jika ada
                if 'detail_url' in image_info:
                    url_parts = image_info['detail_url'].split('/')
                    if url_parts:
                        image_info['asset_id'] = url_parts[-1].split('?')[0]
                
                return image_info
                
        except Exception as e:
            print(f"Error extracting image info: {e}")
            
        return None
    
    def save_to_csv(self, data, filename="adobe_trending_images.csv"):
        """Simpan data ke file CSV"""
        if data:
            df = pd.DataFrame(data)
            df.to_csv(filename, index=False, encoding='utf-8')
            print(f"Data disimpan ke {filename}")
        else:
            print("Tidak ada data untuk disimpan")
    
    def save_to_json(self, data, filename="adobe_trending_images.json"):
        """Simpan data ke file JSON"""
        if data:
            with open(filename, 'w', encoding='utf-8') as f:
                json.dump(data, f, indent=2, ensure_ascii=False)
            print(f"Data disimpan ke {filename}")
        else:
            print("Tidak ada data untuk disimpan")
```

### Fungsi Main untuk Menjalankan Scraper

```python
def main():
    """Fungsi utama untuk menjalankan scraper"""
    scraper = AdobeStockScraper()
    
    print("=== Adobe Stock Trending Images Scraper ===")
    print("PENTING: Pastikan penggunaan sesuai dengan Terms of Service Adobe")
    print()
    
    # Pilihan kategori
    categories = ["photos", "vectors", "illustrations"]
    
    for category in categories:
        print(f"\n--- Scraping {category.upper()} ---")
        
        # Ambil data trending images
        trending_data = scraper.get_trending_images(category=category, limit=20)
        
        if trending_data:
            print(f"Berhasil mengambil {len(trending_data)} {category}")
            
            # Simpan ke file
            scraper.save_to_csv(trending_data, f"adobe_trending_{category}.csv")
            scraper.save_to_json(trending_data, f"adobe_trending_{category}.json")
            
            # Tampilkan sample data
            print("\nContoh data yang ditemukan:")
            for i, item in enumerate(trending_data[:3]):
                print(f"{i+1}. {item.get('description', 'No description')[:50]}...")
        else:
            print(f"Tidak ada data {category} yang berhasil diambil")
        
        # Jeda antar kategori
        time.sleep(2)
```

### Script Lengkap

```python
import requests
from bs4 import BeautifulSoup
import time
import json
import pandas as pd
from urllib.parse import urljoin, urlparse
import os

class AdobeStockScraper:
    def __init__(self):
        self.base_url = "https://stock.adobe.com"
        self.session = requests.Session()
        # Set user agent untuk menghindari blocking
        self.session.headers.update({
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36'
        })
        
    def get_trending_images(self, category="photos", limit=50):
        """
        Mengambil data gambar trending dari Adobe Stock
        
        Args:
            category: jenis konten (photos, vectors, illustrations)
            limit: jumlah maksimal gambar yang diambil
        """
        trending_url = f"{self.base_url}/trending/{category}"
        
        try:
            print(f"Mengakses: {trending_url}")
            response = self.session.get(trending_url, timeout=10)
            response.raise_for_status()
            
            soup = BeautifulSoup(response.content, 'html.parser')
            images_data = []
            
            # Cari container gambar (struktur bisa berubah)
            image_containers = soup.find_all('div', {'data-testid': lambda x: x and 'asset-tile' in x}) or \
                             soup.find_all('div', class_=lambda x: x and 'asset' in x.lower()) or \
                             soup.find_all('img', src=lambda x: x and 'adobe' in x)
            
            print(f"Ditemukan {len(image_containers)} gambar")
            
            for i, container in enumerate(image_containers[:limit]):
                if i % 10 == 0:
                    print(f"Progress: {i}/{min(limit, len(image_containers))}")
                
                image_data = self.extract_image_info(container)
                if image_data:
                    images_data.append(image_data)
                
                # Rate limiting - jangan terlalu cepat
                time.sleep(0.5)
                
            return images_data
            
        except requests.RequestException as e:
            print(f"Error saat mengakses website: {e}")
            return []
    
    def extract_image_info(self, container):
        """Extract informasi dari setiap gambar"""
        try:
            image_info = {}
            
            # Cari tag img
            img_tag = container.find('img') if container.name != 'img' else container
            
            if img_tag:
                # URL thumbnail
                image_info['thumbnail_url'] = img_tag.get('src') or img_tag.get('data-src')
                
                # Alt text sebagai deskripsi
                image_info['description'] = img_tag.get('alt', 'No description')
                
                # Cari link detail
                link_tag = container.find_parent('a') or container.find('a')
                if link_tag:
                    image_info['detail_url'] = urljoin(self.base_url, link_tag.get('href', ''))
                
                # Cari informasi tambahan
                parent = container.find_parent()
                if parent:
                    # Cari download count atau popularity indicator
                    download_info = parent.find(text=lambda x: x and ('download' in x.lower() or 'view' in x.lower()))
                    if download_info:
                        image_info['popularity'] = download_info.strip()
                
                # Extract ID dari URL jika ada
                if 'detail_url' in image_info:
                    url_parts = image_info['detail_url'].split('/')
                    if url_parts:
                        image_info['asset_id'] = url_parts[-1].split('?')[0]
                
                return image_info
                
        except Exception as e:
            print(f"Error extracting image info: {e}")
            
        return None
    
    def save_to_csv(self, data, filename="adobe_trending_images.csv"):
        """Simpan data ke file CSV"""
        if data:
            df = pd.DataFrame(data)
            df.to_csv(filename, index=False, encoding='utf-8')
            print(f"Data disimpan ke {filename}")
        else:
            print("Tidak ada data untuk disimpan")
    
    def save_to_json(self, data, filename="adobe_trending_images.json"):
        """Simpan data ke file JSON"""
        if data:
            with open(filename, 'w', encoding='utf-8') as f:
                json.dump(data, f, indent=2, ensure_ascii=False)
            print(f"Data disimpan ke {filename}")
        else:
            print("Tidak ada data untuk disimpan")

def main():
    """Fungsi utama untuk menjalankan scraper"""
    scraper = AdobeStockScraper()
    
    print("=== Adobe Stock Trending Images Scraper ===")
    print("PENTING: Pastikan penggunaan sesuai dengan Terms of Service Adobe")
    print()
    
    # Pilihan kategori
    categories = ["photos", "vectors", "illustrations"]
    
    for category in categories:
        print(f"\n--- Scraping {category.upper()} ---")
        
        # Ambil data trending images
        trending_data = scraper.get_trending_images(category=category, limit=20)
        
        if trending_data:
            print(f"Berhasil mengambil {len(trending_data)} {category}")
            
            # Simpan ke file
            scraper.save_to_csv(trending_data, f"adobe_trending_{category}.csv")
            scraper.save_to_json(trending_data, f"adobe_trending_{category}.json")
            
            # Tampilkan sample data
            print("\nContoh data yang ditemukan:")
            for i, item in enumerate(trending_data[:3]):
                print(f"{i+1}. {item.get('description', 'No description')[:50]}...")
        else:
            print(f"Tidak ada data {category} yang berhasil diambil")
        
        # Jeda antar kategori
        time.sleep(2)

# Alternative: Menggunakan Adobe Stock API (Recommended)
def adobe_api_example():
    """
    Contoh menggunakan Adobe Stock API (lebih recommended)
    Perlu API key dari Adobe Developer Console
    """
    print("\n=== REKOMENDASI: Gunakan Adobe Stock API ===")
    print("Untuk hasil yang lebih baik dan legal, daftarkan aplikasi di:")
    print("https://developer.adobe.com/console/")
    print()
    print("Contoh endpoint API:")
    print("GET https://stock.adobe.com/Rest/Media/1/Search/Files")
    print("Parameters: locale, search_parameters[words], etc.")
    
    # Contoh implementasi API (memerlukan API key)
    api_example = """
    import requests
    
    def search_adobe_stock_api(api_key, query="trending", limit=50):
        headers = {
            'X-API-Key': api_key,
            'X-Product': 'MySampleApp/1.0'
        }
        
        params = {
            'locale': 'en_US',
            'search_parameters[words]': query,
            'search_parameters[limit]': limit,
            'search_parameters[offset]': 0,
            'result_columns[]': ['id', 'title', 'creator_name', 'thumbnail_url', 'nb_downloads']
        }
        
        response = requests.get(
            'https://stock.adobe.com/Rest/Media/1/Search/Files',
            headers=headers,
            params=params
        )
        
        return response.json()
    """
    
    print("Contoh kode API:")
    print(api_example)

if __name__ == "__main__":
    print("DISCLAIMER:")
    print("- Script ini untuk tujuan edukasi")
    print("- Selalu patuhi robots.txt dan Terms of Service")
    print("- Gunakan rate limiting yang wajar")
    print("- Pertimbangkan menggunakan API resmi Adobe Stock")
    print("-" * 50)
    
    choice = input("Lanjutkan scraping? (y/n): ").lower()
    
    if choice == 'y':
        main()
        adobe_api_example()
    else:
        print("Script dibatalkan")
```

## 🚀 Cara Menjalankan

### 1. Persiapan Environment

```bash
# Buat virtual environment (opsional tapi recommended)
python -m venv adobe_scraper_env

# Aktivasi virtual environment
# Windows:
adobe_scraper_env\Scripts\activate
# Mac/Linux:
source adobe_scraper_env/bin/activate

# Install dependencies
pip install requests beautifulsoup4 pandas lxml
```

### 2. Simpan Script

Simpan kode lengkap di atas dalam file `adobe_scraper.py`

### 3. Jalankan Script

```bash
python adobe_scraper.py
```

### 4. Ikuti Instruksi

Script akan menampilkan disclaimer dan meminta konfirmasi sebelum mulai scraping.

## 📊 Output yang Dihasilkan

### File CSV
Script akan menghasilkan 3 file CSV:
- `adobe_trending_photos.csv`
- `adobe_trending_vectors.csv`
- `adobe_trending_illustrations.csv`

### File JSON
Dan 3 file JSON:
- `adobe_trending_photos.json`
- `adobe_trending_vectors.json`
- `adobe_trending_illustrations.json`

### Struktur Data

Setiap entry akan berisi:
```json
{
  "thumbnail_url": "https://...",
  "description": "Beautiful sunset landscape...",
  "detail_url": "https://stock.adobe.com/...",
  "asset_id": "123456789",
  "popularity": "1.2k downloads"
}
```

## ⚡ Contoh Penggunaan Spesifik

### Scraping Kategori Tertentu

```python
# Hanya scraping photos
scraper = AdobeStockScraper()
photos_data = scraper.get_trending_images(category="photos", limit=30)
scraper.save_to_csv(photos_data, "trending_photos.csv")
```

### Custom Analysis

```python
# Analisis data trending
import pandas as pd

df = pd.read_csv("adobe_trending_photos.csv")

# Lihat deskripsi paling umum
print(df['description'].value_counts().head(10))

# Filter gambar dengan kata kunci tertentu
nature_images = df[df['description'].str.contains('nature|landscape', case=False, na=False)]
print(f"Ditemukan {len(nature_images)} gambar alam")
```

## 🛡️ Best Practices & Ethical Guidelines

### Rate Limiting
```python
# Tambahkan delay lebih lama jika diperlukan
time.sleep(1)  # 1 detik antar request
```

### Respect robots.txt
Cek terlebih dahulu: https://stock.adobe.com/robots.txt

### Headers yang Proper
```python
headers = {
    'User-Agent': 'Educational Research Bot 1.0',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'Accept-Language': 'en-US,en;q=0.5',
    'Accept-Encoding': 'gzip, deflate',
    'Connection': 'keep-alive',
}
```

## 🔄 Alternatif yang Lebih Baik: Adobe Stock API

### Kelebihan API Resmi:
- Data yang lebih akurat dan lengkap
- Rate limits yang jelas dan fair
- Dukungan official dari Adobe
- Tidak melanggar Terms of Service
- Metadata yang lebih rich

### Cara Mendaftar:
1. Kunjungi: https://developer.adobe.com/console/
2. Buat aplikasi baru
3. Dapatkan API key
4. Gunakan endpoint resmi

### Contoh API Request:
```python
import requests

def get_trending_via_api(api_key):
    headers = {
        'X-API-Key': api_key,
        'X-Product': 'YourAppName/1.0'
    }
    
    params = {
        'locale': 'en_US',
        'search_parameters[order]': 'relevance',
        'search_parameters[limit]': 50,
        'result_columns[]': [
            'id', 'title', 'creator_name', 
            'thumbnail_url', 'nb_downloads',
            'keywords', 'category'
        ]
    }
    
    response = requests.get(
        'https://stock.adobe.com/Rest/Media/1/Search/Files',
        headers=headers,
        params=params
    )
    
    return response.json()
```

## 🐛 Troubleshooting

### Error: SSL Certificate
```python
# Jika ada masalah SSL
import ssl
ssl._create_default_https_context = ssl._create_unverified_context
```

### Error: Timeout
```python
# Tingkatkan timeout
response = self.session.get(trending_url, timeout=30)
```

### Error: Blocked by Website
```python
# Ganti user agent atau tambahkan proxy
headers = {
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36'
}
```

## 📝 Catatan Penting

1. **Website Structure Changes**: Adobe bisa mengubah struktur website kapan saja, sehingga script perlu disesuaikan
2. **Legal Compliance**: Pastikan scraping sesuai dengan terms of service dan hukum yang berlaku
3. **Educational Purpose**: Script ini dibuat untuk pembelajaran, bukan untuk tujuan komersial
4. **API is Better**: Sangat disarankan menggunakan Adobe Stock API untuk kebutuhan serius

## 📚 Resources Tambahan

- [Adobe Stock API Documentation](https://developer.adobe.com/stock/docs/)
- [Beautiful Soup Documentation](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)
- [Requests Documentation](https://docs.python-requests.org/)
- [Web Scraping Ethics](https://blog.apify.com/web-scraping-ethics/)

---

**Happy Learning! �
