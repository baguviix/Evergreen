<div align="center">

# 🌲 Evergreen ILS
### Highly-Scalable Open-Source Integrated Library System
*With Complete & Standardized Georgian (ka-GE) Localization*

[![License: GPL v2](https://img.shields.io/badge/License-GPLv2-blue.svg)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.html)
[![Language: Georgian](https://img.shields.io/badge/Localization-ka--GE%20100%25-green.svg)](#-georgian-localization--ქართული-ლოკალიზაცია)
[![Evergreen ILS](https://img.shields.io/badge/Evergreen-Library%20System-darkgreen.svg)](https://evergreen-ils.org/)
[![Status](https://img.shields.io/badge/Status-Active-brightgreen.svg)]()

<p align="center">
  <a href="#-about-evergreen">About</a> •
  <a href="#-georgian-localization--ქართული-ლოკალიზაცია">Georgian Localization</a> •
  <a href="#-key-features">Key Features</a> •
  <a href="#-tech-stack">Tech Stack</a> •
  <a href="#-installation--documentation">Documentation</a>
</p>

---

</div>

## 📖 About Evergreen

**Evergreen** is an enterprise-grade, highly scalable **Integrated Library System (ILS)** originally developed for the Georgia Public Library Service (PINES). Today, it powers thousands of public, academic, consortial, and special libraries worldwide.

Evergreen helps library staff manage cataloging, circulation, acquisitions, serials, and materials sharing, while providing library patrons with a modern, feature-rich public access catalog (OPAC).

---

## 🇬🇪 Georgian Localization / ქართული ლოკალიზაცია

ამ რეპოზიტორიაში სრულად არის ინტეგრირებული და განახლებული **Evergreen ILS-ის ქართული ლოკალიზაცია (ka-GE)**, რომელიც შეესაბამება თანამედროვე ქართულ სალიტერატურო ენის ნორმებსა და სტანდარტულ საბიბლიოთეკო ტერმინოლოგიას.

### 📊 ლოკალიზაციის სტატისტიკა
* **ფაილები:** `51` PO ფაილი (`ka-GE.po`)
* **მოცულობა:** `97,600+` ხაზი
* **სტატუსი:** 100% ვალიდირებული სინტაქსი

### 📚 ძირითადი საბიბლიოთეკო ტერმინოლოგია
| ტერმინი (ILS) | წინა თარგმანი | განახლებული სტანდარტი |
| :--- | :--- | :--- |
| **Copy / Item** | ასლი / ნივთი | **ეგზემპლარი / ერთეული** |
| **Patron** | პატრონი / მფარველი | **მკითხველი** |
| **Hold** | შეკავება / დაჭერა | **დაჯავშნა / ჯავშანი** |
| **Circulation** | ტირაჟი / მიმოქცევა | **ცირკულაცია (გაცემა/დაბრუნება)** |
| **Call Number** | ზარის ნომერი | **საბიბლიოთეკო შიფრი** |
| **Bucket** | თაიგული / ვედრო | **კალათა** |
| **Purchase Order** | შესყიდვის ორდერი | **შესყიდვის შეკვეთა** |
| **Lineitem** | ხაზის ერთეული | **შეკვეთის პოზიცია** |
| **Authority Record** | უფლებამოსილების ჩანაწერი | **ავტორიტეტული ჩანაწერი** |

---

## ✨ Key Features

- **🚀 Powerful Circulation Engine:** Flexible policies for loan durations, recurring fines, transit routing, hold queues, and offline operations.
- **📚 Cataloging & MARC Control:** Full MARC21/UNIMARC support, authority control, Z39.50 copy importation, and batch record editing (Vandelay).
- **🔍 Public Access Catalog (OPAC):** Modern, mobile-responsive catalog with faceted browsing, cover art, bookbags, and patron self-service.
- **💼 Acquisitions & Invoicing:** Complete fiscal tracking, vendor management, purchase orders, EDI interfaces, and fund administration.
- **📰 Serials Management:** Check-in, predictive enumeration/chronology, routing lists, and MARC holdings (MFHD) integration.
- **📈 Advanced Analytics:** Intuitive web-based report builder with scheduler, template repository, and export capabilities (CSV, Excel).

---

## 🛠 Tech Stack

Evergreen is built on a distributed, message-passing client-server architecture:

```
[ Web Client / OPAC ]  <-->  [ Apache Web Server ]
                                     |
                             [ OpenSRF Router ]
                                     |
                 +-------------------+-------------------+
                 |                   |                   |
            [ Perl APIs ]       [ C Engines ]      [ Node/Python ]
                 +-------------------+-------------------+
                                     |
                          [ PostgreSQL Database ]
```

* **Core Services:** [OpenSRF](https://evergreen-ils.org/opensrf/) (Open Service Request Framework)
* **Database:** PostgreSQL (with PL/pgSQL, full-text search, and trigger rules)
* **Server Side:** Perl, C, Jabber/XMPP
* **Client / UI:** Angular, JavaScript, Bootstrap, Dojo / XUL

---

## 🚀 Installation & Documentation

Comprehensive installation, administration, and development guides are maintained in the official documentation:

* 📄 **Official Website:** [evergreen-ils.org](https://evergreen-ils.org/)
* 📚 **Documentation:** [Evergreen Documentation Hub](https://docs.evergreen-ils.org/)
* ⚙️ **Server Installation:** See `docs/modules/installation/pages/server_installation.adoc`
* 💬 **Community & Support:** [Evergreen Mailing Lists & IRC](https://evergreen-ils.org/communicate/)

---

## 📄 License

Evergreen is free software, distributed under the terms of the **GNU General Public License (GPL) version 2** or later. See the [LICENSE.txt](LICENSE.txt) file for details.
