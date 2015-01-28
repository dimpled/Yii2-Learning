# การติดตั้ง Yii 2
การติดตั้ง Yii 2 นั้นหลักๆ ทำได้ 2 วิธีครับ คือ
* ติดตั้งผ่าน Composer
* ติดตั้งโดยดาวน์โหลดไฟล์มาติดตั้งเอง จาก http://www.yiiframework.com/download/

## ติดตั้งผ่าน Composer
เป็นวิธีที่แนะนำเพราะสามารถทำได้ง่ายๆ รายละเอียด Composer มารถดูได้ที่นี่  [getcomposer.org](https://getcomposer.org/download/)  การติดตั้งสำหรับ linux และ Mac OS X ให้คุณพิมพ์คำสั่ง

```
curl -s http://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
```
อธิบายคำสั่ง บรรทัดแรกเป็นการดาวน์โหลดไฟล์ติดตั้ง "composer.phar" บรรทัดที่สองเป็นการก๊อบปี้ไฟล์ "composer.phar" ไปไว้ที่ /usr/local/bin เพื่อให้สามารถเรียกใช้งานได้โดยไม่ต้องระบบุ path

ในส่วนของ Windows คุณสามารถดาวน์โหลดไฟล์ไปติดตั้งได้เลย [Composer-Setup.exe](https://getcomposer.org/Composer-Setup.exe) และสามารถอ่านรายละเอียดการใช้งานเพิ่มเติมได้ที่นี่ [Composer Documentation](https://getcomposer.org/doc/) ไม่ว่าจะเป็นการติดตั้งหรือปัญหาการใช้งานต่างๆ

หลังจากติดตั้ง Composer เสร็จให้คุณรันคำสั่งด้านล่างนี้เพื่อทำการอัพเดทตัว Composer
```
composer self-update
```
เมื่อติดตั้งเสร็จแล้วคุณสามารถติดตั้ง Yii โดยใช้คำสั่งนี้
```
composer global require "fxp/composer-asset-plugin:1.0.0"
composer create-project --prefer-dist yiisoft/yii2-app-basic basic
```
"basic" บรรทัดสุดท้ายคือชื่อ app
> *ในตัวอย่างนี้ผมติดตั้งไว้ที่ /Applications/MAMP/htdocs/ ในตัวจำลอง server

หลังจากติดตั้งเสร็จให้ทดสอบเข้าใช้งาน http://localhost/basic/web/index.php

## ติดตั้งโดยดาวโหลดไฟล์มาติดตั้งเอง
