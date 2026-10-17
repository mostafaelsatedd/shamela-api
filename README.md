# 🕌 المكتبة الإسلامية - دلائل AI

واجهة برمجية شاملة للتعامل مع المكتبة الشاملة (shamela.ws)

## 📋 المتطلبات

- Node.js v18+
- npm أو yarn

## 🚀 التثبيت والتشغيل

```bash
# 1. تثبيت المتطلبات
npm install

# 2. اختبار الاتصال بـ API الشاملة
npm test

# 3. تشغيل الخادم
npm start
```

## 📡 API Endpoints

### الأساسية

| المسار | الوصف |
|--------|-------|
| `GET /api/categories` | قائمة الأقسام |
| `GET /api/authors` | قائمة المؤلفين |
| `GET /api/books` | قائمة الكتب |
| `GET /api/books/:id` | تفاصيل كتاب |
| `GET /api/books/:id/content` | محتوى كتاب |
| `GET /api/search?q=...` | البحث |
| `GET /api/stats` | الإحصائيات |

### المزامنة

| المسار | الوصف |
|--------|-------|
| `POST /api/sync/master` | مزامنة الأقسام والكتب والمؤلفين |
| `POST /api/sync/book/:id` | تحميل/تحديث كتاب معين |

## 🔧 أمثلة الاستخدام

### 1. مزامنة البيانات الأساسية (أول مرة)

```bash
curl -X POST http://localhost:3001/api/sync/master
```

**الرد:**
```json
{
  "success": true,
  "message": "تم التحديث",
  "version": 5,
  "stats": {
    "categories": 35,
    "books": 8552,
    "authors": 2145
  }
}
```

### 2. البحث في الكتب

```bash
curl "http://localhost:3001/api/search?q=صحيح البخاري"
```

**الرد:**
```json
{
  "success": true,
  "query": "صحيح البخاري",
  "results": {
    "books": [
      {
        "id": 1681,
        "name": "صحيح البخاري",
        "author": "البخاري",
        "type": "book"
      }
    ]
  }
}
```

### 3. جلب قائمة الكتب مع فلترة

```bash
# كل الكتب
curl "http://localhost:3001/api/books"

# كتب قسم معين
curl "http://localhost:3001/api/books?category=5"

# كتب مؤلف معين
curl "http://localhost:3001/api/books?author=123"

# البحث في العناوين
curl "http://localhost:3001/api/books?search=تفسير"
```

### 4. تحميل كتاب

```bash
curl -X POST http://localhost:3001/api/sync/book/1681
```

**الرد:**
```json
{
  "success": true,
  "message": "تم التحديث",
  "major": 2,
  "minor": 1,
  "pageCount": 4500,
  "titleCount": 350
}
```

### 5. قراءة محتوى كتاب

```bash
curl "http://localhost:3001/api/books/1681/content"
```

**الرد:**
```json
{
  "success": true,
  "data": {
    "pages": [
      {
        "id": 1,
        "part": "1",
        "page": 1,
        "content": "بسم الله الرحمن الرحيم..."
      }
    ],
    "titles": [
      {
        "id": 1,
        "content": "كتاب بدء الوحي",
        "pageId": 5,
        "children": [
          {
            "id": 2,
            "content": "باب كيف كان بدء الوحي",
            "pageId": 6,
            "children": []
          }
        ]
      }
    ],
    "totalPages": 4500,
    "totalTitles": 350
  }
}
```

## 📁 هيكل الملفات

```
shamela_test/
├── package.json          # المتطلبات
├── test_shamela.js       # سكريبت الاختبار
├── server.js             # خادم API
├── README.md             # التوثيق
├── shamela_data/         # البيانات المحفوظة
│   ├── master.json       # الأقسام والكتب والمؤلفين
│   └── books/            # محتوى الكتب
│       ├── 1681.json     # صحيح البخاري
│       └── ...
└── shamela_cache/        # الإصدارات والملفات المؤقتة
    ├── master_version.json
    └── book_1681_version.json
```

## 🔄 آلية العمل

### 1. جلب Master (أول مرة)

```
┌──────────────┐     GET /patches/master?version=0     ┌──────────────┐
│   العميل     │ ─────────────────────────────────────▶ │  الشاملة API │
│              │                                        │              │
│              │ ◀─────────────────────────────────────│              │
└──────────────┘     { patch_url, Version: 5 }         └──────────────┘
        │
        │ تحميل ZIP
        ▼
┌──────────────┐
│   ZIP File   │
│ ┌──────────┐ │
│ │book.db   │ │ ─── SQLite ───▶ 8500 كتاب
│ │author.db │ │ ─── SQLite ───▶ 2100 مؤلف
│ │category.db│ │ ─── SQLite ───▶ 35 قسم
│ └──────────┘ │
└──────────────┘
```

### 2. جلب كتاب

```
┌──────────────┐  GET /patches/book-updates/1681?major=0&minor=0  ┌──────────────┐
│   العميل     │ ───────────────────────────────────────────────▶ │  الشاملة API │
│              │                                                   │              │
│              │ ◀─────────────────────────────────────────────── │              │
└──────────────┘  { major_release_url, minor_release_url, ... }    └──────────────┘
        │
        │ تحميل Major ZIP
        ▼
┌──────────────┐
│   ZIP File   │
│ ┌──────────┐ │
│ │ page.db  │ │ ─── SQLite ───▶ صفحات الكتاب
│ │ title.db │ │ ─── SQLite ───▶ فهرس الكتاب
│ └──────────┘ │
└──────────────┘
```

## ⚠️ ملاحظات مهمة

1. **الـ API ليس للبحث المباشر**: يُرجع ملفات SQLite مضغوطة وليس JSON
2. **يجب مزامنة Master أولاً**: قبل استخدام أي وظيفة
3. **الكتب تُحمّل حسب الطلب**: لتوفير المساحة
4. **نظام الإصدارات**: يتتبع التحديثات لتجنب إعادة التحميل

## 🔗 روابط مفيدة

- [المكتبة الشاملة](https://shamela.ws)
- [تطبيق تراث](https://app.turath.io)
- [توثيق API الشاملة](https://dev.shamela.ws)
تحديث تجريبي لنشر Railway

## 📄 الترخيص

MIT - للاستخدام في مشروع دلائل AI
