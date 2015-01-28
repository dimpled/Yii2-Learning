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
บรรทัดแรกเป็นการดาวน์โหลดไฟล์ติดตั้ง "composer.phar" บรรทัดที่สองเป็นการนำไฟล์ "composer.phar" ไปไว้ที่ bin เพื่อให้สามารถเรียกใช้งานได้โดยไม่ต้องระบบุ path

ในส่วนของ Windows คุณสามารถดาวน์โหลดไฟล์ไปติดตั้งได้เลย [Composer-Setup.exe](https://getcomposer.org/Composer-Setup.exe) อ่านรายละเอียดการใช้งานเพิ่มเติมได้ที่นี่ [Composer Documentation](https://getcomposer.org/doc/) ไม่ว่าจะเป็นการติดตั้งหรือปัญหาการใช้งานต่างๆ 

หลังจากติดตั้งเสร็จให้คุณรันคำสั่งนี้เพื่อทำการอัพเดท Composer
```
composer self-update
```

## ติดตั้งโดยดาวโหลดไฟล์มาติดตั้งเอง
