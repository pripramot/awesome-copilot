---
name: thai-development
description: 'Development patterns, best practices, and code examples optimized for Thai developers and Thai-language applications. Covers Thai text handling, locale configuration, Thai business logic patterns, and culturally appropriate UX guidelines.'
---

# Thai Development (การพัฒนาสำหรับแอปพลิเคชันไทย)

Best practices and patterns for building software applications in Thai context.

## Thai Text Handling (การจัดการข้อความภาษาไทย)

### Unicode and Encoding

```python
# Always use UTF-8 encoding for Thai text
text = "สวัสดีครับ ยินดีต้อนรับ"

# File operations with Thai content
with open("output.txt", "w", encoding="utf-8") as f:
    f.write(text)

# String operations work normally with Thai
length = len(text)  # Counts characters correctly
words = text.split()  # Word boundary splitting
```

### Thai Word Segmentation

```python
# Using PyThaiNLP for accurate Thai word segmentation
from pythainlp.tokenize import word_tokenize

text = "ฉันรักประเทศไทยมากที่สุด"
words = word_tokenize(text, engine="newmm")
# Result: ['ฉัน', 'รัก', 'ประเทศไทย', 'มาก', 'ที่สุด']

# For search and indexing
from pythainlp.tokenize import sent_tokenize
sentences = sent_tokenize(paragraph)
```

### Thai Date and Time Formatting

```python
from datetime import datetime
import locale

# Thai Buddhist Era (พ.ศ.) conversion
def to_thai_year(gregorian_year: int) -> int:
    return gregorian_year + 543

def format_thai_date(dt: datetime) -> str:
    thai_months = [
        "มกราคม", "กุมภาพันธ์", "มีนาคม", "เมษายน",
        "พฤษภาคม", "มิถุนายน", "กรกฎาคม", "สิงหาคม",
        "กันยายน", "ตุลาคม", "พฤศจิกายน", "ธันวาคม"
    ]
    day = dt.day
    month = thai_months[dt.month - 1]
    year = to_thai_year(dt.year)
    return f"{day} {month} พ.ศ. {year}"

# Usage
now = datetime.now()
print(format_thai_date(now))  # e.g., "12 มีนาคม พ.ศ. 2568"
```

### Thai Number Formatting

```python
def format_thai_number(amount: float, currency: str = "บาท") -> str:
    """Format numbers in Thai style with currency."""
    formatted = f"{amount:,.2f}"
    return f"{formatted} {currency}"

def thai_number_words(n: int) -> str:
    """Convert number to Thai words (for checks, formal documents)."""
    # Simplified example
    units = ["", "หนึ่ง", "สอง", "สาม", "สี่", "ห้า", "หก", "เจ็ด", "แปด", "เก้า"]
    # Full implementation would handle all Thai number conventions
    pass
```

## Thai Business Logic Patterns

### VAT Calculation (ภาษีมูลค่าเพิ่ม)

```python
VAT_RATE = 0.07  # 7% VAT in Thailand

def calculate_vat(price: float) -> dict:
    vat_amount = price * VAT_RATE
    total = price + vat_amount
    return {
        "price_before_vat": price,
        "vat_amount": round(vat_amount, 2),
        "total": round(total, 2),
        "vat_rate": f"{VAT_RATE * 100:.0f}%"
    }

def extract_vat(price_including_vat: float) -> dict:
    """Extract VAT from a price that already includes VAT."""
    price_ex_vat = price_including_vat / (1 + VAT_RATE)
    vat = price_including_vat - price_ex_vat
    return {
        "price_before_vat": round(price_ex_vat, 2),
        "vat_amount": round(vat, 2),
        "total": price_including_vat
    }
```

### Thai ID Card Validation (ตรวจสอบบัตรประชาชน)

```python
def validate_thai_id(id_number: str) -> bool:
    """Validate Thai National ID using checksum algorithm."""
    if len(id_number) != 13 or not id_number.isdigit():
        return False

    total = sum(
        int(id_number[i]) * (13 - i)
        for i in range(12)
    )
    check_digit = (11 - (total % 11)) % 10
    return check_digit == int(id_number[12])
```

### Thai Phone Number Formatting

```python
import re

def format_thai_phone(phone: str) -> str:
    """Format Thai phone numbers consistently."""
    digits = re.sub(r'\D', '', phone)

    # Mobile numbers (08x, 09x, 06x)
    if len(digits) == 10 and digits[0] == '0':
        return f"{digits[:3]}-{digits[3:6]}-{digits[6:]}"

    # Landline (02x, 03x, etc.)
    if len(digits) == 9 and digits[0] == '0':
        return f"{digits[:2]}-{digits[2:5]}-{digits[5:]}"

    return phone  # Return as-is if format not recognized
```

## UI/UX Guidelines for Thai Apps

### Font Recommendations

```css
/* Thai-optimized font stack */
body {
  font-family:
    'Sarabun',         /* Google Fonts - modern Thai */
    'Noto Sans Thai',  /* Google Fonts - comprehensive */
    'Thonburi',        /* macOS built-in */
    'TH Sarabun New',  /* Windows built-in */
    sans-serif;
}

/* Ensure proper line-height for Thai script */
p, li {
  line-height: 1.8;  /* Thai characters need more space */
}
```

### Responsive Text Sizing

```css
/* Thai text often needs slightly larger minimum sizes */
body {
  font-size: 16px;
  min-font-size: 14px;
}

.thai-content {
  word-break: break-word;  /* Handle long Thai words */
  overflow-wrap: break-word;
}
```

## Localization (การแปลภาษา)

### i18n Setup with Thai Support

```javascript
// i18n configuration for Thai (th-TH)
const messages = {
  th: {
    greeting: 'สวัสดี {name}',
    items_count: '{count} รายการ | {count} รายการ',
    date_format: 'DD MMMM YYYY',
  },
  en: {
    greeting: 'Hello {name}',
    items_count: '{count} item | {count} items',
    date_format: 'MMMM DD, YYYY',
  }
}
```

## Best Practices Summary

1. **Always UTF-8** - ใช้ UTF-8 encoding ทุกครั้งที่จัดการข้อความไทย
2. **Buddhist Era** - แสดงปีเป็น พ.ศ. ในหน้าจอที่ผู้ใช้มองเห็น
3. **Word segmentation** - ใช้ PyThaiNLP หรือ library ที่เหมาะสมสำหรับ search/NLP
4. **Font selection** - เลือก font ที่รองรับภาษาไทยได้ดีและ line-height ที่เหมาะสม
5. **Input validation** - ตรวจสอบ Thai-specific inputs (บัตรประชาชน, เบอร์โทร)
6. **VAT compliance** - ใส่การคำนวณ VAT 7% ในระบบราคาทุกระบบ
