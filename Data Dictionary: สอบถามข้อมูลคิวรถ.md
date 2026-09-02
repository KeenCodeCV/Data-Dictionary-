# Data Dictionary: สอบถามข้อมูลคิวรถ

เอกสารนี้อธิบายคอลัมน์ที่แสดงในหน้า **สอบถามข้อมูลคิวรถ** ของเว็บ โดยแยกตามประเภทการสอบถามและเรียงตามลำดับคอลัมน์บนหน้าจอจริง ครอบคลุมคอลัมน์หลัก 4 ประเภท รวมถึงตารางรายละเอียดเอกสารของคิวจ่าย

สถานะข้อมูล ณ วันที่ **2 กันยายน 2026**

## 1. ขอบเขตและหลักฐาน

| ประเภทการสอบถาม | โปรแกรมเดิม | แหล่งข้อมูลกริด | จำนวนคอลัมน์ |
|---|---|---|---:|
| คิวรับสินค้า | `FMTKQUEI100` | `FEED.FM_QUEUE_TIME_VIEW` | 31 |
| คิวจ่ายสินค้า | `FMTKQUEI200` | `FEED.FM_QUEUE_TIME_ISSUED_VIEW` | 14 |
| คิวอื่น ๆ | `FMTKQUEI300` | `FEED.FM_QUEUE_TIME_VIEW` | 13 |
| สอบถามข้อมูลเวลาคิวรถ | `FMTKQUEI400` | `FEED.FM_QUEUE_TIME_VIEW` | 16 |

ชุดคอลัมน์ตรวจเทียบจากหลักฐานต่อไปนี้:

- ลำดับ `MASREP_SELECT` ของ default layout ใน `STD.MAS_REPORT_LAYOUT`
- หัวคอลัมน์ภาษาไทยใน `STD.MAS_COLUMN_DEFINITION` (`LANGUAGE_CODE='THA'`)
- นิยาม `FEED.FM_QUEUE_TIME_VIEW` และ `FEED.FM_QUEUE_TIME_ISSUED_VIEW` บน SWTUAT
- ชุดคอลัมน์ที่ API อนุญาตใน `QueueInquiryController.ModeColumns`
- ภาพและพฤติกรรมหน้าจอระบบเดิม โดยเฉพาะคิวอื่น ๆ ซึ่งยืนยันว่าไม่มี `WEIGHT_NO`

> คำว่า “หัวคอลัมน์” ในตารางด้านล่างหมายถึงข้อความที่ระบบเดิมและเว็บแสดงจริง ไม่ใช่คำแปลที่ตั้งขึ้นใหม่ หาก `MAS_COLUMN_DEFINITION` ไม่มีคำแปล ระบบจะแสดงชื่อ field ตรงตัว เช่น `MANAGE_CODE`

## 2. แหล่งข้อมูลหลักและกติกาที่ใช้ร่วมกัน

### 2.1 ตารางหลักและ master

| แหล่งข้อมูล | หน้าที่ |
|---|---|
| `FEED.FM_QUEUE` | ข้อมูลคิวหลัก เช่น เลขคิว ทะเบียน วันเวลา น้ำหนัก ผลวิเคราะห์ และสถานะ |
| `STD.MAS_VENDOR_GENERAL` | ชื่อผู้ขายสำหรับคิวรับและคิวอื่น ๆ |
| `STD.MAS_CUSTOMER_GENERAL` / `MAS_CUSTOMER_GENERAL` | ชื่อลูกค้าสำหรับคิวจ่าย |
| `FEED.FM_MAS_PRODUCT_GROUP` | ชื่อกลุ่มสินค้า |
| `FEED.GD2_FM_ANALYZE` | คำอธิบายผลวิเคราะห์ |
| `FEED.FM_ORDER_HEADER` | วันที่อนุมัติคำสั่งซื้อ/คำสั่งงาน |
| `FEED.FM_MATERIAL_DOC_HEADER`, `FEED.FM_STOCK` | วันที่รับเอกสาร โอนย้าย และเสร็จสิ้น |
| `FEED.FM_QCSAMPLING_ST` | รหัสปัญหาและแนวทางจัดการที่บันทึกจากงาน QC |
| `FEED.GD2_FM_PROBLEM*` | คำอธิบายปัญหาย่อย ปัญหารอง และปัญหาหลัก |
| `FEED.GD2_FM_QCMANAGE*` | คำอธิบายการจัดการและการจัดการหลัก |

### 2.2 การแสดงค่า

- คอลัมน์ชนิด `DATE` เก็บทั้งวันที่และเวลา เว็บแสดงเป็น `DD/MM/YYYY HH:mm` เมื่อมีส่วนเวลา
- ค่า `NULL` ของทะเบียน รหัสวิเคราะห์ และบาง field ถูก View แปลงเป็นช่องว่างด้วย `NVL`/`TRIM`
- แถวที่ `CANCEL_FLAG='Y'` ไม่ถูกนำมาแสดง
- `FM_QUEUE_TIME_VIEW` จำกัดข้อมูลย้อนหลังไว้ประมาณ 730 วันตามนิยาม View
- ตัวกรอง “คิวสินค้า” เปรียบเทียบกับ `QUEUE_GROUP_REF` แต่คอลัมน์ “คิวสินค้า” ในโหมดเวลาแสดงค่า `QUEUE_GROUP` ทั้งสอง field จึงไม่ควรถูกตีความว่าเป็น field เดียวกัน

## 3. คิวรับสินค้า (`receive` / `FMTKQUEI100`)

แหล่งข้อมูลกริด: `FEED.FM_QUEUE_TIME_VIEW` โดยกรอง `QUEUE_TYPE='01'`

| # | Column ID | หัวคอลัมน์ | ชนิดข้อมูล | ความหมายของข้อมูล | ต้นทาง/สูตรใน View |
|---:|---|---|---|---|---|
| 1 | `QUEUE_NO_SHOW` | เลขที่คิว | `VARCHAR2(86)` | เลขคิวแบบที่หน้าสอบถามใช้ ประกอบด้วยสถานะ เลขรันนิง และลำดับย่อย | `FM_QUEUE.QUEUE_STATUS \|\| TO_CHAR(QUEUE_NO,'0000') \|\| '-' \|\| QUEUE_SEQ` เช่น `0 0001-0`; ช่องว่างก่อนเลขเกิดจากรูปแบบตัวเลขของ Oracle |
| 2 | `LICENSE_PLATE` | ทะเบียนรถ | `VARCHAR2(20)` | ทะเบียนรถหัวลาก/รถหลัก | `FM_QUEUE.LICENSE_PLATE`; ถ้าไม่มีค่า View คืนช่องว่าง |
| 3 | `TRAILER_LICENSE` | ทะเบียนรถพ่วง | `VARCHAR2(20)` | ทะเบียนพ่วง หากคิวมีรถพ่วง | `TRIM(NVL(FM_QUEUE.TRAILER_LICENSE,''))`; field นี้ยังใช้ตัดสินการนับรถพ่วงเพิ่มอีก 1 คันใน KPI |
| 4 | `PRODUCT_GROUP` | กลุ่มสินค้า | `VARCHAR2(18)` | รหัสกลุ่มสินค้าหรือกลุ่มวัตถุดิบของคิว | `FM_QUEUE.PRODUCT_GROUP` |
| 5 | `PRODUCT_GROUP_DESC_LOC` | ชื่อกลุ่มสินค้า | `VARCHAR2(300)` | ชื่อภาษาไทยของกลุ่มสินค้า | `FM_MAS_PRODUCT_GROUP.DESC_LOC` โดย join `FM_QUEUE.PRODUCT_GROUP = FM_MAS_PRODUCT_GROUP.PRODUCT_GROUP` |
| 6 | `VENDOR_CODE` | รหัสผู้ขายปลายทาง | `VARCHAR2(18)` | รหัสผู้ขาย/คู่ค้าที่นำสินค้าเข้า | `FM_QUEUE.VENDOR_CODE` |
| 7 | `VENDOR_NAME_LOCAL` | ชื่อผู้ขาย | `VARCHAR2(300)` | ชื่อภาษาไทยของผู้ขาย/คู่ค้า | สำหรับ `QUEUE_TYPE='01'` ใช้ `STD.MAS_VENDOR_GENERAL.NAME_LOC` โดย join จาก `FM_QUEUE.VENDOR_CODE` |
| 8 | `ORIGIN_QTY` | ปริมาณต้นทาง | `NUMBER(9,0)` | ปริมาณที่ระบุจากต้นทางก่อนเข้าสู่กระบวนการชั่ง หน่วยขึ้นกับประเภทสินค้า/การตั้งค่าธุรกิจ | `FM_QUEUE.ORIGIN_QTY` โดยตรง ไม่มีการแปลงหน่วยใน View |
| 9 | `ANALYZE_DESC_LOC` | ผลวิเคราะห์ | `VARCHAR2(100)` | คำอธิบายผลวิเคราะห์ภาษาไทย เช่น ผ่าน ไม่ผ่าน หรือช่วยรับ | `GD2_FM_ANALYZE.DESC_LOC` โดย join จาก `FM_QUEUE.ANALYZE_CODE`; ถ้าไม่พบคืนช่องว่าง |
| 10 | `ARRIVE_DATE` | เวลารถเข้า | `DATE` | วันที่และเวลาที่รถมาถึง/เข้าสู่จุดเริ่มคิว | `FM_QUEUE.ARRIVE_DATE` |
| 11 | `CHECK_IN_DATE` | เวลาชั่งเข้า | `DATE` | วันที่และเวลาชั่งครั้งเข้า | `FM_QUEUE.CHECK_IN_DATE` |
| 12 | `CHECK_OUT_DATE` | เวลาชั่งออก | `DATE` | วันที่และเวลาชั่งครั้งออก | `FM_QUEUE.CHECK_OUT_DATE` |
| 13 | `DEPART_DATE` | เวลารถออก | `DATE` | วันที่และเวลาที่รถออกจากโรงงาน | `FM_QUEUE.DEPART_DATE` |
| 14 | `PRODUCT_CODE1` | เบอร์วัตถุดิบ | `VARCHAR2(18)` | รหัส/เบอร์วัตถุดิบของรถหลักที่บันทึกกับคิว | `FM_QUEUE.PRODUCT_CODE1` |
| 15 | `MOISTURE1` | ความชื้น | `NUMBER(5,2)` | ค่าความชื้นของวัตถุดิบรถหลัก | `FM_QUEUE.MOISTURE1`; View ไม่เติมหน่วยหรือคำนวณเพิ่ม |
| 16 | `FAT` | FAT | `NUMBER(5,2)` | ค่า FAT ที่บันทึกจากการตรวจวิเคราะห์ | `FM_QUEUE.FAT`; View ไม่เติมหน่วยหรือคำนวณเพิ่ม |
| 17 | `BGYF1` | BGYF | `VARCHAR2(20)` | ค่า BGYF ของวัตถุดิบรถหลักตามที่ผู้ปฏิบัติงาน/ระบบวิเคราะห์บันทึก | `FM_QUEUE.BGYF1`; schema ไม่มีคำอธิบายหน่วย จึงต้องยึดค่าที่บันทึกโดยไม่ตีความเพิ่ม |
| 18 | `PRODUCT_CODE2` | เบอร์วัตถุดิบ (รถพ่วง) | `VARCHAR2(18)` | รหัส/เบอร์วัตถุดิบของรถพ่วง | `FM_QUEUE.PRODUCT_CODE2` |
| 19 | `MOISTURE2` | ความชื้น (รถพ่วง) | `NUMBER(5,2)` | ค่าความชื้นของวัตถุดิบรถพ่วง | `FM_QUEUE.MOISTURE2` |
| 20 | `BGYF2` | BGYF (รถพ่วง) | `VARCHAR2(20)` | ค่า BGYF ของวัตถุดิบรถพ่วง | `FM_QUEUE.BGYF2`; schema ไม่มีคำอธิบายหน่วย |
| 21 | `REMARK_DESC1` | หมายเหตุรอบแรก | `VARCHAR2(300)` | หมายเหตุจากการตรวจ/ดำเนินงานรอบแรก | `FM_QUEUE.REMARK_DESC1` |
| 22 | `WEIGHT_NO` | เลขที่บัตรชั่ง | `NUMBER(15,0)` | เลขอ้างอิงบัตรชั่งของคิว | `FM_QUEUE.WEIGHT_NO` |
| 23 | `CHECK_NET_WGH` | น้ำหนักสุทธิ | `NUMBER` | น้ำหนักสุทธิของคิวรับ | สูตรคิวรับ `FM_QUEUE.CHECK_IN_WGH - FM_QUEUE.CHECK_OUT_WGH` |
| 24 | `SERIAL_NO` | Serial Number | `NUMBER(15,0)` | รหัสอ้างอิงภายในที่ไม่ซ้ำของรายการคิว ใช้เชื่อมข้อมูลข้ามตาราง | `FM_QUEUE.SERIAL_NO`; ไม่ใช่เลขคิวที่ผู้ใช้รับหน้าจุดลงทะเบียน |
| 25 | `ANALYZE_CODE` | ผลวิเคราะห์ | `VARCHAR2(3)` | รหัสดิบของผลวิเคราะห์ ใช้กำหนดสถานะและสี | `NVL(FM_QUEUE.ANALYZE_CODE,' ')`; ต่างจาก `ANALYZE_DESC_LOC` ซึ่งเป็นข้อความจาก master |
| 26 | `ANALYZE_DATE` | เวลาบันทึกผลรอบแรก | `DATE` | วันเวลาที่บันทึกผลวิเคราะห์ โดยให้ความสำคัญกับผลรอบแรก | `NVL(FM_QUEUE.ANALYZE_DATE_1ST, FM_QUEUE.ANALYZE_DATE)` |
| 27 | `PROBLEM_NAME` | ปัญหาย่อย | `VARCHAR2(300)` | คำอธิบายปัญหาที่เลือกในงาน QC | `GD2_FM_PROBLEM.DESC_LOC` โดย `FM_QCSAMPLING_ST.PROBLEM_CODE` join กับ master |
| 28 | `PROBLEM_SUB` | ปัญหารอง | `VARCHAR2(300)` | คำอธิบายหมวดปัญหาระดับรอง | `GD2_FM_PROBLEM_SUB.DESC_LOC` โดย join จาก `FM_QCSAMPLING_ST.PROBLEM_SUB_CODE` |
| 29 | `PROBLEM_MAIN` | ปัญหาหลัก | `VARCHAR2(300)` | คำอธิบายหมวดปัญหาระดับหลัก | `GD2_FM_PROBLEM_MAIN.DESC_LOC` โดย join จาก `FM_QCSAMPLING_ST.PROBLEM_MAIN_CODE` |
| 30 | `MANAGE_CODE` | MANAGE_CODE | `VARCHAR2(300)` | **คำอธิบาย** วิธีจัดการที่เลือกในงาน QC แม้ชื่อ field และหัวคอลัมน์จะดูเหมือนเป็นรหัส | View เลือก `GD2_FM_QCMANAGE.DESC_LOC AS MANAGE_CODE` โดย join จาก `FM_QCSAMPLING_ST.MANAGE_CODE`; ไม่มีคำแปลใน `MAS_COLUMN_DEFINITION` จึงแสดงหัว `MANAGE_CODE` ตามระบบเดิม |
| 31 | `MANAGE_MAIN` | การจัดการหลัก | `VARCHAR2(300)` | คำอธิบายแนวทางจัดการระดับหลัก | `GD2_FM_QCMANAGE_MAIN.DESC_LOC` โดย join จาก `FM_QCSAMPLING_ST.MANAGE_MAIN_CODE` |

## 4. คิวจ่ายสินค้า (`issue` / `FMTKQUEI200`)

แหล่งข้อมูลกริด: `FEED.FM_QUEUE_TIME_ISSUED_VIEW` โดย View กรอง `QUEUE_TYPE='02'` ไว้แล้ว

| # | Column ID | หัวคอลัมน์ | ชนิดข้อมูล | ความหมายของข้อมูล | ต้นทาง/สูตรใน View |
|---:|---|---|---|---|---|
| 1 | `QUEUE_NO_SHOW` | เลขที่คิว | `VARCHAR2(86)` | เลขคิวสำหรับหน้าสอบถาม ประกอบด้วยสถานะ เลขรันนิง และลำดับย่อย | `FM_QUEUE.QUEUE_STATUS \|\| TO_CHAR(QUEUE_NO,'0000') \|\| '-' \|\| QUEUE_SEQ` |
| 2 | `LICENSE_PLATE` | ทะเบียนรถ | `VARCHAR2(20)` | ทะเบียนรถหลักที่มารับสินค้า | `NVL(FM_QUEUE.LICENSE_PLATE,' ')` |
| 3 | `TRAILER_LICENSE` | ทะเบียนรถพ่วง | `VARCHAR2(20)` | ทะเบียนรถพ่วง หากมี | `TRIM(NVL(FM_QUEUE.TRAILER_LICENSE,''))`; ใช้นับรถพ่วงเพิ่มใน KPI ด้วย |
| 4 | `PRODUCT_GROUP` | กลุ่มสินค้า | `VARCHAR2(18)` | รหัสกลุ่มสินค้าที่จ่าย | `FM_QUEUE.PRODUCT_GROUP` |
| 5 | `PRODUCT_GROUP_DESC_LOC` | ชื่อกลุ่มสินค้า | `VARCHAR2(300)` | ชื่อภาษาไทยของกลุ่มสินค้าที่จ่าย | `FM_MAS_PRODUCT_GROUP.DESC_LOC` โดย join จาก `FM_QUEUE.PRODUCT_GROUP` |
| 6 | `VENDOR_CODE` | รหัสผู้ขายปลายทาง | `VARCHAR2(18)` | รหัสลูกค้า/ปลายทางของคิวจ่าย ชื่อ field และหัวคอลัมน์ยังคงคำว่า vendor ตาม legacy | `FM_QUEUE.VENDOR_CODE` แต่ join ชื่อด้วย `MAS_CUSTOMER_GENERAL.CUSTOMER_CODE` |
| 7 | `VENDOR_NAME_LOCAL` | ชื่อผู้ขาย | `VARCHAR2(300)` | ชื่อลูกค้าหรือปลายทางภาษาไทย แม้หัวคอลัมน์ระบบเดิมใช้คำว่า “ชื่อผู้ขาย” | `MAS_CUSTOMER_GENERAL.NAME_LOC` โดย join `FM_QUEUE.VENDOR_CODE = CUSTOMER_CODE` |
| 8 | `WEIGHT_NO` | เลขที่บัตรชั่ง | `NUMBER(15,0)` | เลขอ้างอิงบัตรชั่งของคิวจ่าย | `FM_QUEUE.WEIGHT_NO` |
| 9 | `ARRIVE_DATE` | เวลารถเข้า | `DATE` | วันที่และเวลารถมาถึงโรงงาน | `FM_QUEUE.ARRIVE_DATE` |
| 10 | `CHECK_IN_DATE` | เวลาชั่งเข้า | `DATE` | วันที่และเวลาชั่งครั้งเข้า | `FM_QUEUE.CHECK_IN_DATE` |
| 11 | `CHECK_OUT_DATE` | เวลาชั่งออก | `DATE` | วันที่และเวลาชั่งครั้งออก | `FM_QUEUE.CHECK_OUT_DATE` |
| 12 | `DEPART_DATE` | เวลารถออก | `DATE` | วันที่และเวลารถออกจากโรงงาน | `FM_QUEUE.DEPART_DATE` |
| 13 | `QTIME_CHECK` | เวลาชั่ง(แตกต่าง) | `VARCHAR2(82)` | ระยะเวลาระหว่างชั่งเข้าและชั่งออก แสดงเป็น `ชั่วโมง:นาที` | คำนวณจาก `FM_QUEUE.CHECK_OUT_DATE - FM_QUEUE.CHECK_IN_DATE`; ถ้าวันเวลาใดไม่มี ค่าอาจว่าง |
| 14 | `SERIAL_NO` | Serial Number | `NUMBER(15,0)` | รหัสอ้างอิงภายในของคิว ใช้เปิดรายละเอียด invoice/สินค้า | `FM_QUEUE.SERIAL_NO` |

> `FM_QUEUE_TIME_ISSUED_VIEW` ไม่มีชุดคอลัมน์ปัญหา/การจัดการที่ใช้ในคิวรับ และไม่มี `ANALYZE_DATE` ดังนั้นห้ามนำ mapping ของคิวรับมาใช้กับคิวจ่ายโดยอัตโนมัติ

## 5. คิวอื่น ๆ (`other` / `FMTKQUEI300`)

แหล่งข้อมูลกริด: `FEED.FM_QUEUE_TIME_VIEW` โดยกรอง `QUEUE_TYPE='03'`

| # | Column ID | หัวคอลัมน์ | ชนิดข้อมูล | ความหมายของข้อมูล | ต้นทาง/สูตรใน View |
|---:|---|---|---|---|---|
| 1 | `QUEUE_NO_SHOW` | เลขที่คิว | `VARCHAR2(86)` | เลขคิวสำหรับหน้าสอบถาม ประกอบด้วยสถานะ เลขรันนิง และลำดับย่อย | `FM_QUEUE.QUEUE_STATUS \|\| TO_CHAR(QUEUE_NO,'0000') \|\| '-' \|\| QUEUE_SEQ` |
| 2 | `LICENSE_PLATE` | ทะเบียนรถ | `VARCHAR2(20)` | ทะเบียนรถหลัก | `NVL(FM_QUEUE.LICENSE_PLATE,' ')` |
| 3 | `TRAILER_LICENSE` | ทะเบียนรถพ่วง | `VARCHAR2(20)` | ทะเบียนรถพ่วง หากมี | `TRIM(NVL(FM_QUEUE.TRAILER_LICENSE,''))`; ใช้นับรถพ่วงเพิ่มใน KPI ด้วย |
| 4 | `PRODUCT_GROUP` | กลุ่มสินค้า | `VARCHAR2(18)` | รหัสกลุ่มสินค้าของคิวอื่น ๆ | `FM_QUEUE.PRODUCT_GROUP` |
| 5 | `PRODUCT_GROUP_DESC_LOC` | ชื่อกลุ่มสินค้า | `VARCHAR2(300)` | ชื่อภาษาไทยของกลุ่มสินค้า | `FM_MAS_PRODUCT_GROUP.DESC_LOC` โดย join จาก `FM_QUEUE.PRODUCT_GROUP` |
| 6 | `VENDOR_CODE` | รหัสผู้ขายปลายทาง | `VARCHAR2(18)` | รหัสคู่ค้า/ผู้เกี่ยวข้องที่บันทึกกับคิว | `FM_QUEUE.VENDOR_CODE` |
| 7 | `VENDOR_NAME_LOCAL` | ชื่อผู้ขาย | `VARCHAR2(300)` | ชื่อภาษาไทยของคู่ค้า/ผู้เกี่ยวข้อง | สำหรับ `QUEUE_TYPE='03'` ใช้ `STD.MAS_VENDOR_GENERAL.NAME_LOC` |
| 8 | `ANALYZE_DESC_LOC` | ผลวิเคราะห์ | `VARCHAR2(100)` | คำอธิบายผลวิเคราะห์ภาษาไทย | `GD2_FM_ANALYZE.DESC_LOC` โดย join จาก `FM_QUEUE.ANALYZE_CODE` |
| 9 | `ARRIVE_DATE` | เวลารถเข้า | `DATE` | วันที่และเวลารถมาถึง/เข้าโรงงาน | `FM_QUEUE.ARRIVE_DATE` |
| 10 | `CHECK_IN_DATE` | เวลาชั่งเข้า | `DATE` | วันที่และเวลาชั่งครั้งเข้า | `FM_QUEUE.CHECK_IN_DATE` |
| 11 | `DEPART_DATE` | เวลารถออก | `DATE` | วันที่และเวลารถออกจากโรงงาน | `FM_QUEUE.DEPART_DATE` |
| 12 | `CHECK_OUT_DATE` | เวลาชั่งออก | `DATE` | วันที่และเวลาชั่งครั้งออก | `FM_QUEUE.CHECK_OUT_DATE` |
| 13 | `SERIAL_NO` | Serial Number | `NUMBER(15,0)` | รหัสอ้างอิงภายในที่ไม่ซ้ำของรายการคิว | `FM_QUEUE.SERIAL_NO` |

> คิวอื่น ๆ **ไม่มีคอลัมน์ `WEIGHT_NO` (เลขที่บัตรชั่ง)** ตามหน้าจอ legacy ที่ผู้ใช้ใช้งานจริง แม้ default layout บน SWTUAT จะเคยพบคอลัมน์นี้ เว็บจึงยึด behavioral spec ของระบบเดิมและไม่แสดง field ดังกล่าว

## 6. สอบถามข้อมูลเวลาคิวรถ (`time` / `FMTKQUEI400`)

แหล่งข้อมูลกริด: `FEED.FM_QUEUE_TIME_VIEW` ประเภทคิวมาจากค่าที่ผู้ใช้เลือก

| # | Column ID | หัวคอลัมน์ | ชนิดข้อมูล | ความหมายของข้อมูล | ต้นทาง/สูตรใน View |
|---:|---|---|---|---|---|
| 1 | `QUEUE_NO_SHOW` | เลขที่คิว | `VARCHAR2(86)` | เลขคิวสำหรับหน้าสอบถาม ประกอบด้วยสถานะ เลขรันนิง และลำดับย่อย | `FM_QUEUE.QUEUE_STATUS \|\| TO_CHAR(QUEUE_NO,'0000') \|\| '-' \|\| QUEUE_SEQ` |
| 2 | `QUEUE_GROUP` | คิวสินค้า | `VARCHAR2(15)` | รหัสกลุ่มคิวที่รายการถูกจัดอยู่จริง | `FM_QUEUE.QUEUE_GROUP`; **ไม่ใช่** `QUEUE_GROUP_REF` ที่ตัวกรองคิวสินค้าใช้ค้นหา |
| 3 | `PRODUCT_GROUP_DESC_LOC` | ชื่อกลุ่มสินค้า | `VARCHAR2(300)` | ชื่อภาษาไทยของกลุ่มสินค้าหรือวัตถุดิบ | `FM_MAS_PRODUCT_GROUP.DESC_LOC` โดย join จาก `FM_QUEUE.PRODUCT_GROUP` |
| 4 | `LICENSE_PLATE` | ทะเบียนรถ | `VARCHAR2(20)` | ทะเบียนรถหลัก | `NVL(FM_QUEUE.LICENSE_PLATE,' ')` |
| 5 | `TRAILER_LICENSE` | ทะเบียนรถพ่วง | `VARCHAR2(20)` | ทะเบียนรถพ่วง หากมี | `TRIM(NVL(FM_QUEUE.TRAILER_LICENSE,''))` |
| 6 | `VENDOR_CODE` | รหัสผู้ขายปลายทาง | `VARCHAR2(18)` | รหัสคู่ค้าของคิว | `FM_QUEUE.VENDOR_CODE`; ความหมายเป็นผู้ขายในคิวรับ/อื่น ๆ และเป็นลูกค้าปลายทางในคิวจ่าย |
| 7 | `VENDOR_NAME_LOCAL` | ชื่อผู้ขาย | `VARCHAR2(300)` | ชื่อคู่ค้าภาษาไทยตามชนิดคิว | View ใช้ `STD.MAS_VENDOR_GENERAL.NAME_LOC` สำหรับชนิด `01`/`03` และ `STD.MAS_CUSTOMER_GENERAL.NAME_LOC` สำหรับชนิด `02` |
| 8 | `ARRIVE_DATE` | เวลารถเข้า | `DATE` | จุดเริ่มต้นเวลา: รถมาถึง/เข้าสู่โรงงาน | `FM_QUEUE.ARRIVE_DATE` |
| 9 | `CHECK_IN_DATE` | เวลาชั่งเข้า | `DATE` | วันที่และเวลาชั่งครั้งเข้า | `FM_QUEUE.CHECK_IN_DATE` |
| 10 | `APPROVE_DATE` | วันที่อนุมัติ | `DATE` | วันเวลาอนุมัติเอกสารคำสั่งที่ผูกกับคิว | ค่าต่ำสุดของ `NVL(FM_ORDER_HEADER.APPROVE_DATE, FM_ORDER_HEADER.ORD_DOC_DATE)` ต่อคิว |
| 11 | `RECEIVE_DATE` | วันที่รับเอกสาร | `DATE` | วันเวลาแรกที่รับเอกสารวัสดุของคิว | `MIN(FM_MATERIAL_DOC_HEADER.RECEIVE_DATE)` ต่อ `PLANT_CODE + SERIAL_NO` |
| 12 | `TRANSFER_DATE` | เวลาโอนย้าย | `DATE` | วันเวลาแรกที่โอนย้ายเอกสาร/สต็อก | ใช้ `MIN(FM_MATERIAL_DOC_HEADER.TRANSFER_DATE)`; หากไม่มีใช้ `MIN(FM_STOCK.CREATE_DATE)` ของเอกสารอ้างอิงประเภทสต็อก `147` |
| 13 | `FINISHED_DATE` | เวลาเสร็จ | `DATE` | วันเวลาล่าสุดที่งานเอกสารวัสดุของคิวเสร็จ | `MAX(FM_MATERIAL_DOC_HEADER.FINISHED_DATE)` ต่อ `PLANT_CODE + SERIAL_NO` |
| 14 | `CHECK_OUT_DATE` | เวลาชั่งออก | `DATE` | วันที่และเวลาชั่งครั้งออก | `FM_QUEUE.CHECK_OUT_DATE` |
| 15 | `DEPART_DATE` | เวลารถออก | `DATE` | วันที่และเวลารถออกจากโรงงาน | `FM_QUEUE.DEPART_DATE` |
| 16 | `QTIME` | เวลาอยู่ในโรงงาน | `VARCHAR2(82)` | เวลารวมตั้งแต่รถเข้าจนรถออก แสดงเป็น `ชั่วโมง:นาที` และชั่วโมงอาจเกิน 24 | ถ้ามี `DEPART_DATE` ใช้ `DEPART_DATE - ARRIVE_DATE`; ถ้ายังไม่ออกใช้ `SYSDATE - ARRIVE_DATE` จึงเปลี่ยนตามเวลาปัจจุบัน |

## 7. รายละเอียดคิวจ่าย (Drill-down)

เมื่อดับเบิลคลิกคิวจ่าย ระบบใช้ `SERIAL_NO` เปิดรายละเอียด เส้นทางข้อมูลขึ้นกับ `FM_MAS_ORG_CONFIG.MULTI_SIZE`

### 7.1 ชั้นที่ 1: รายการ Invoice (`MULTI_SIZE='Y'`)

| Column ID | หัวคอลัมน์ | ความหมายของข้อมูล | ต้นทาง/สูตร |
|---|---|---|---|
| `INVOICE_NO` | เลขที่ Invoice | เลขที่เอกสารขาย/Invoice ที่เชื่อมกับคิว | `FM_SALE_HEADER.SAL_DOC_NO`; เชื่อมผ่านเอกสารวัสดุของ `PLANT_CODE + SERIAL_NO` |
| `CUSTOMER_NAME` | ชื่อลูกค้า | รหัสลูกค้าต่อด้วยชื่อภาษาไทย | `FM_SALE_HEADER.CUSTOMER_CODE \|\| '-' \|\| MAS_CUSTOMER_GENERAL.NAME_LOC` |

### 7.2 ชั้นที่ 2: รายการสินค้าใน Invoice (`MULTI_SIZE='Y'`)

| Column ID | หัวคอลัมน์ | ความหมายของข้อมูล | ต้นทาง/สูตร |
|---|---|---|---|
| `PRODUCT_CODE` | รหัสสินค้า | รหัสสินค้าที่ขาย; แถวสรุปใช้ข้อความ “รวมทั้งหมด” | `FM_SALE_DETAIL.PRODUCT_CODE` |
| `PRODUCT_NAME` | ชื่อสินค้า | ชื่อสินค้าไทย | `MAS_PRODUCT_GENERAL.DESC_LOC` |
| `GRADE_CODE` | เกรด | รหัสเกรดสินค้า; ค่า `00` ถูกแสดงเป็นช่องว่าง | `FM_SALE_DETAIL.GRADE_CODE` ผ่าน `DECODE` |
| `MEDICINE_CODE` | ยา | รหัสยา/สูตรยาที่ผูกกับสินค้า; ค่า `000` ถูกแสดงเป็นช่องว่าง | `FM_SALE_DETAIL.MEDICINE_CODE` ผ่าน `DECODE` |
| `STOCK_QTY_LARGE` | Master | จำนวนหน่วยบรรจุหลัก | `Convert_Size_Large(PRODUCT_CODE, SUM(FM_SALE_DETAIL.SALE_QTY))` |
| `STOCK_QTY_MINOR` | Inner | จำนวนหน่วยย่อยภายใน | `Convert_Size_Minor(PRODUCT_CODE, SUM(FM_SALE_DETAIL.SALE_QTY))` |
| `INVOICE_NO` | เลขที่ Invoice | เลข Invoice ของรายการสินค้า | `TO_CHAR(FM_SALE_HEADER.SAL_DOC_NO)`; แถวรวมทั้งหมดเป็นช่องว่าง |

### 7.3 รายการสินค้าโดยตรง (`MULTI_SIZE<>'Y'`)

โรงงานที่ไม่ใช้ multi-size จะเปิดรายละเอียดสินค้าโดยตรงและไม่มีชั้น Invoice แยก

| Column ID | หัวคอลัมน์ | ความหมายของข้อมูล | ต้นทาง/สูตร |
|---|---|---|---|
| `PRODUCT_CODE` | รหัสสินค้า | รหัสสินค้าจากรายละเอียดเอกสารวัสดุ | `FM_MATERIAL_DOC_DETAIL.PRODUCT_CODE` |
| `PRODUCT_NAME` | ชื่อสินค้า | ชื่อสินค้าไทย | `MAS_PRODUCT_GENERAL.DESC_LOC` |
| `GRADE_CODE` | เกรด | รหัสเกรด; ค่า `00` ถูกแสดงเป็นช่องว่าง | `FM_MATERIAL_DOC_DETAIL.GRADE_CODE` ผ่าน `DECODE` |
| `MEDICINE_CODE` | ยา | รหัสยา/สูตรยา; ค่า `000` ถูกแสดงเป็นช่องว่าง | `FM_MATERIAL_DOC_DETAIL.MEDICINE_CODE` ผ่าน `DECODE` |
| `PICKING_QTY` | จำนวนจ่าย | ผลรวมปริมาณที่เบิก/จ่าย | `SUM(NVL(FM_MATERIAL_DOC_DETAIL.PICKING_QTY,0))` |
| `UNLOAD_QTY` | จำนวนลง | ผลรวมปริมาณลงสินค้า; มีในผลชั้นแรกของเส้นทาง non-multi-size | `SUM(NVL(FM_MATERIAL_DOC_DETAIL.UNLOAD_QTY,0))` |
| `INVOICE_NO` | เลขที่ Invoice | เลขที่เอกสารขายที่เชื่อมกับเอกสารวัสดุ หากมี | `FM_SALE_HEADER.SAL_DOC_NO` ผ่าน outer join |

## 8. จุดที่มักตีความผิด

1. `QUEUE_NO_SHOW` ไม่ใช่ `QUEUE_NO` ดิบ และไม่ใช่เลขที่คิวรูปแบบเดียวกับหน้าบันทึกคิว
2. `SERIAL_NO` คือรหัสเชื่อมรายการภายใน ไม่ใช่เลขคิวสำหรับผู้ใช้งาน
3. `PRODUCT_GROUP` คือรหัสกลุ่มสินค้า ส่วน `QUEUE_GROUP` คือรหัสกลุ่มคิว และตัวกรองคิวสินค้าใช้ `QUEUE_GROUP_REF`
4. `ANALYZE_CODE` เป็นรหัส แต่ `ANALYZE_DESC_LOC` เป็นข้อความจาก master แม้หัวทั้งคู่ใช้คำว่า “ผลวิเคราะห์”
5. `MANAGE_CODE` ในผลลัพธ์ View เป็นข้อความ `DESC_LOC` ไม่ใช่รหัสดิบ แต่ต้องคงหัว `MANAGE_CODE` เพื่อให้ตรงระบบเดิม
6. `VENDOR_CODE`/`VENDOR_NAME_LOCAL` ของคิวจ่ายแทนลูกค้า/ปลายทาง แม้ชื่อ field และหัวคอลัมน์ยังใช้คำว่าผู้ขาย
7. `CHECK_NET_WGH` เปลี่ยนทิศทางการลบตามชนิดคิว: คิวรับใช้ชั่งเข้า - ชั่งออก ส่วนชนิดอื่นใช้ชั่งออก - ชั่งเข้า
8. `QTIME` เป็นค่าที่เปลี่ยนได้เองขณะรถยังไม่ออก เพราะคำนวณเทียบกับ `SYSDATE`

## 9. เอกสารอ้างอิง

- `docs/LEGACY_QUEUE_INQUIRY_ANALYSIS_TH.md` โดยเฉพาะ §19.8, §19.9, §20.6 และ §20.11
- `docs/QUEUE_INQUIRY_CHARACTERIZATION_TESTS_TH.md`
- `queueautoapi/Controllers/QueueInquiryController.cs`
- `queueautofront/pages/queue/queryqueue.vue`

