# DataForThai Scraper Workflow (n8n)

## ภาพรวม (Overview)

Workflow นี้ถูกออกแบบมาเพื่อดึงข้อมูล (Scrape) ของบริษัทต่างๆ จากเว็บไซต์ `dataforthai.com` โดยอัตโนมัติ ผู้ใช้สามารถใส่รายชื่อ URL ของบริษัทที่ต้องการได้หลายรายการ จากนั้น Workflow จะเข้าไปดึงข้อมูลรายละเอียดของแต่ละบริษัท แล้วรวบรวมข้อมูลทั้งหมดสร้างเป็นไฟล์ CSV เพื่อให้สามารถดาวน์โหลดไปใช้งานต่อได้

## ขั้นตอนการทำงานของ Workflow (Workflow Steps)

Workflow ทำงานตามลำดับขั้นตอนดังนี้:

### 1. **Start Workflow**
*   **ประเภท Node:** `Manual Trigger`
*   **หน้าที่:** เป็นปุ่มเริ่มต้นการทำงานทั้งหมดของ Workflow ผู้ใช้สามารถกดปุ่ม "Execute Workflow" เพื่อเริ่มกระบวนการได้ทันที

### 2. **Visit Homepage to Get Cookies**
*   **ประเภท Node:** `HTTP Request`
*   **หน้าที่:** ขั้นตอนนี้สำคัญมากในการหลีกเลี่ยงระบบป้องกันบอท โดยจะทำการเข้าชมหน้าแรกของ `https://www.dataforthai.com/` เพื่อรับ Session และ Cookies ที่ถูกต้องจากเซิร์ฟเวอร์ ทำให้การส่งคำขอในขั้นตอนถัดไปดูเหมือนเป็นการใช้งานจากเบราว์เซอร์จริง

### 3. **Input URLs Here**
*   **ประเภท Node:** `Code`
*   **หน้าที่:** Node นี้ทำหน้าที่เป็นแหล่งข้อมูลของ URL ทั้งหมดที่ต้องการดึงข้อมูล ผู้ใช้สามารถแก้ไขโค้ดใน Node นี้เพื่อเพิ่ม, ลบ, หรือเปลี่ยนแปลงรายการ URL ของบริษัทได้โดยตรง จากนั้นโค้ดจะทำการประมวลผลและส่งออกเป็นรายการ URL เพื่อให้ Node ถัดไปทำงานแบบวนลูป (Loop)

### 4. **Scrape Each URL**
*   **ประเภท Node:** `HTTP Request`
*   **หน้าที่:** Node นี้จะทำงานแบบวนลูป โดยรับ URL มาทีละรายการจาก Node ก่อนหน้า และใช้ Cookie ที่ได้จากขั้นตอนที่ 2 ในการส่งคำขอ (Request) ไปยังหน้าของบริษัทนั้นๆ เพื่อดึงเนื้อหา HTML ของหน้าเว็บกลับมา

### 5. **Extract Company Details**
*   **ประเภท Node:** `HTML`
*   **หน้าที่:** Node นี้คือหัวใจของการแยกข้อมูล ทำหน้าที่เหมือน `driver.find_element` ในโค้ด Python โดยจะรับโค้ด HTML ที่ได้จาก Node `Scrape Each URL` มาทำการแยกข้อมูลตามเงื่อนไขที่กำหนดไว้
*   **การตั้งค่า:**
    *   **Source Data**: `Input Field` (รับข้อมูล HTML จาก Node ก่อนหน้าโดยอัตโนมัติ)
    *   **Operation**: `Extract HTML Content`
    *   **Extraction Values**: ใช้ CSS Selector หรือ XPath เพื่อระบุตำแหน่งของข้อมูลที่ต้องการดึงออกมา ดังตารางนี้:

| Key (ชื่อข้อมูล) | CSS Selector / XPath ที่ใช้ดึง |
| :--- | :--- |
| `company_name_th` | `div#maindata > h2` |
| `company_name_en` | `div#maindata > h3` |
| `registration_number`| `td:contains('เลขทะเบียน') + td` |
| `business_type` | `td:contains('ประกอบธุรกิจ') + td` |
| `status` | `td:contains('สถานะ') + td` |
| `registration_date` | `td:contains('วันที่จดทะเบียน') + td` |
| `registered_capital`| `td:contains('ทุนจดทะเบียน') + td` |
| `address` | `td:contains('ที่ตั้ง') + td > a` |

### 6. **Create CSV File**
*   **ประเภท Node:** `Spreadsheet File`
*   **หน้าที่:** เป็นขั้นตอนสุดท้ายของ Workflow โดยจะรวบรวมข้อมูลทั้งหมดที่ถูกดึงและแยกออกมาในขั้นตอนก่อนหน้า แล้วแปลงให้อยู่ในรูปแบบของไฟล์ `.csv` เมื่อ Workflow ทำงานเสร็จสิ้น ผู้ใช้จะสามารถดาวน์โหลดไฟล์นี้จากหน้าผลลัพธ์ (Output) ของ Node นี้ได้ทันที

## วิธีการใช้งาน (How to Use)

1.  **นำเข้า Workflow:** นำไฟล์ JSON ที่มีอยู่ไป Import เข้าสู่ n8n ของคุณ
2.  **แก้ไขรายการ URL:** ไปที่ Node `Input URLs Here` และแก้ไขรายการ URL ภายในเครื่องหมาย ``` `` ``` ตามที่ต้องการ
3.  **สั่งให้ Workflow ทำงาน:** กดปุ่ม **Execute Workflow** ที่มุมขวาบน
4.  **ดาวน์โหลดผลลัพธ์:** เมื่อ Workflow ทำงานเสร็จสิ้น ให้ไปที่ Node สุดท้าย (`Create CSV File`) และกดที่แท็บ **Output** เพื่อดาวน์โหลดไฟล์ CSV ที่ได้