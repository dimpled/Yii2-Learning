# การติดตั้ง Yii 2
การติดตั้ง Yii 2 นั้นหลักๆ ทำได้ 2 วิธีครับ คือ
* ติดตั้งผ่าน Composer
* ติดตั้งโดยดาวน์โหลดไฟล์มาติดตั้งเอง จาก http://www.yiiframework.com/download/

## ติดตั้งผ่าน Composer
เป็นวิธีที่แนะนำ เราสามารถดูรายละเอียด Composer เพิ่มเติมได้ที่นี่  [getcomposer.org](https://getcomposer.org/download/)  การติดตั้งสำหรับ linux และ Mac OS X ให้คุณพิมพ์คำสั่ง

```
curl -s http://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
```
อธิบายคำสั่ง บรรทัดแรกเป็นการดาวน์โหลดไฟล์ติดตั้ง "composer.phar" บรรทัดที่สองเป็นการก๊อบปี้ไฟล์ "composer.phar" ไปไว้ที่ /usr/local/bin เพื่อให้สามารถเรียกใช้งานได้โดยไม่ต้องระบบุ path

ในส่วนของ Windows คุณสามารถดาวน์โหลดไฟล์ไปติดตั้งได้เลย [Composer-Setup.exe](https://getcomposer.org/Composer-Setup.exe) และสามารถอ่านรายละเอียดการใช้งานเพิ่มเติมได้ที่นี่ [Composer Documentation](https://getcomposer.org/doc/) ไม่ว่าจะเป็นการติดตั้งหรือปัญหาการใช้งานต่างๆ

หลังจากติดตั้ง Composer เสร็จให้คุณรันคำสั่งด้านล่างนี้เพื่อทำการอัพเดทตัว Composer:
```
composer self-update
```
ในส่วนของการติดตั้ง Yii มี 2 แบบคือ basic และ advance

### แบบ Basic
 คุณสามารถติดตั้งแบบ basic โดยใช้คำสั่งนี้:

```
composer global require "fxp/composer-asset-plugin:1.0.0"
composer create-project --prefer-dist yiisoft/yii2-app-basic basic
```
"basic" บรรทัดสุดท้ายคือชื่อ app
> Note:ในตัวอย่างนี้ผมติดตั้งไว้ที่ /Applications/MAMP/htdocs/ ในตัวจำลอง server

หลังจากติดตั้งเสร็จให้ทดสอบเข้าใช้งาน http://localhost/basic/web/index.php

### แบบ Advance

คุณสามารถติดตั้งแบบ advance โดยใช้คำสั่งนี้:

```
composer global require "fxp/composer-asset-plugin:1.0.0"
composer create-project --prefer-dist yiisoft/yii2-app-advanced yii-application
```
> Note:ในตัวอย่างนี้ผมติดตั้งไว้ที่ /Applications/MAMP/htdocs/ ในตัวจำลอง server

หลังจากสร้างเสร็จจะไม่สามารถใช้งานได้เหมือนแบบ basic จะต้องรันคำสั่ง initialize เพื่อให้สามารถใช้งานได้

1.ให้ cd เข้าไปที่พาทของ application ที่เราได้ทำการสร้างไว้ แล้วรันคำสั่งนี้ :
```
php /path/to/yii-application/init
```

จะพบคำสั่งนี้ ให้เราตอบ 0 เลือกโหมดกำลังพัฒนา
```
Yii Application Initialization Tool v1.0

Which environment do you want the application to be initialized in?

  [0] Development
  [1] Production

  Your choice [0-1, or "q" to quit] 0
```
2.ให้ทำการสร้างดาต้าเบสว่างๆ แล้วคอนฟิกดาต้าเบสที่ไฟล์  common/config/main-local.php

3.หลังจากนั้นให้รันคำสั่ง php yii migrate เพื่อสร้างตารางและระบบจะถาม
```
Yii Migration Tool (based on Yii v2.0.2)

Creating migration history table "migration"...Done.
Total 1 new migration to be applied:
  m130524_201442_init

Apply the above migration? (yes|no) [no]:
```
ให้เราตอบ yes

4.ทดลองเข้าใช้งานที่

frontend : http://localhost/yii-application/frontend/web/index.php

backend : http://localhost/yii-application/backend/web/index.php

## ติดตั้งโดยดาวโหลดไฟล์มาติดตั้งเอง

1. ดาวน์โหลดไฟล์จาก [http://www.yiiframework.com/download/](http://www.yiiframework.com/download/)
2. แตกซิบไฟล์ออกมาแล้วนำไปวางที่ htdocs ของเรา
3. ทำการแก้ไขไฟล์ config/web.php แล้วมองหาบรรทัดที่มีคำว่า cookieValidationKey ให้เราใส่ค่ารหัสลับเข้าไปอะไรก็ได้ เช่น
```
'cookieValidationKey' => 'enter your secret key here',
```
4. จากนั้นทำการทดสอบเข้าใช้งาน

## ปัญหาที่พบบ่อยๆ ในการติดตั้ง
--'
