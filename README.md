import requests
from bs4 import BeautifulSoup
import csv
import os

URL = "https://cafef.vn/du-lieu/bao-cao-tai-chinh/vic/bsheet/2024/0/0/0/bao-cao-tai-chinh-.chn"

def parse_div_rows(html):
    soup = BeautifulSoup(html, "lxml")
    # tìm container có thể chứa báo cáo — điều chỉnh selector nếu cần
    # Ví dụ: tất cả hàng có class chứa 'row' hoặc 'item'
    rows = soup.select(".table tr") or soup.select(".row") or soup.select(".item")
    # nếu vẫn rỗng, lấy tất cả <tr>
    if not rows:
        rows = soup.find_all("tr")
    data = []
    for r in rows:
        cells = r.find_all(["td", "th", "div", "span"])
        values = [c.get_text(strip=True) for c in cells if c.get_text(strip=True) != ""]
        if values:
            data.append(values)
    return data

def save_csv(data, fname="vic_po.csv"):
    os.makedirs(os.path.dirname(fname) or ".", exist_ok=True)
    with open(fname, "w", newline="", encoding="utf-8-sig") as f:
        writer = csv.writer(f)
        for row in data:
            writer.writerow(row)
    print("Saved", fname)

def main():
    headers = {"User-Agent": "Mozilla/5.0"}
    r = requests.get(URL, headers=headers, timeout=15)
    r.raise_for_status()
    data = parse_div_rows(r.text)
    if not data:
        print("Không tìm thấy dữ liệu bằng phương pháp div parsing.")
        return
    save_csv(data)

if __name__ == "__main__":
    main()
