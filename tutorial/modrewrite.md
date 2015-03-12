# เปิดการใช้งาน Pretty urls (mod_rewrite)
Pretty urls ก็คือการทำให้ url ที่มันจำยากๆ ให้มันสามารถอ่านเข้าใจได้ ซึ่งปกติคนทั่วไป ถ้าดูแล้วอาจจะงงๆ นอกจากคนที่อยู่ในสาย develop ก็คงจะพอเข้าใจ แต่ประโยชน์จริงๆ ของมันก็คือ ให้ Search Engine เข้าใจและจำ url ของเราได้ง่าย และยังเป็นประโยชน์กับการทำ SEO อีกด้วย หลายคนอาจจะยังงงๆ ดูตัวอย่างกันเลยดีกว่า T T'

### แบบ url ปกติ
```html
http://www.domain.me/index.php?r=site/index
```
ถ้าดูตาม url เราจะเข้าใจได้ว่า url นี้ รับค่าตัวแปรชื่อ  `r` ซึ่งเก็บค่า `site/index`

### แบบเปิดใช้งาน Pretty Urls
```html
http://www.domain.me/site/index
```
ถ้าสังเกตจะเห็นว่า มีคำว่า `index.php` และตัวแประ `r` หายไป เพราะ mod_rewrite ซ่อนไว้ และ url จะดูสั้นขึ้นมากๆ


## เปิดใช้งาน
ก่อนอื่นให้ทำการคอนฟิกเพื่อเปิดการใช้งาน UrlManager ไปที` config/web.ph` ใส่โค้ดนี้เข้าไป
* `ShowScriptName` คือ ให้แสดง index.php เราเซ็ตเป็น `false`  เพื่อใม่ให้มันแสดง
* `ennablePrettyUrl` คือเปิดการใช้งาน เซ็ตเป็น `true` เพื่อเปิดการใช้งาน
```php
//.....
'urlManager' => [
       'class' => 'yii\web\UrlManager',
       // Disable index.php
       'showScriptName' => false,
       // Disable r= routes
       'enablePrettyUrl' => true,
       'rules' => array(
               '<controller:\w+>/<id:\d+>' => '<controller>/view',
               '<controller:\w+>/<action:\w+>/<id:\d+>' => '<controller>/<action>',
               '<controller:\w+>/<action:\w+>' => '<controller>/<action>',
       ),
],
//......
```

## สร้างไฟล์ .htaccess
สร้างไฟล์ชื่อ `.htaccess` ไว้ที่ `web/.htaccess` แล้วใส่โค้ดตามด้านล่างนี้
```
RewriteEngine on
# If a directory or a file exists, use it directly
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
# Otherwise forward it to index.php
RewriteRule . index.php
```

ทดลองใช้งาน
```
http://www.domain.me/site/index
```

> หากยังไม่ได้ลองเช็คว่าเปิด `AllowOverride All` ที่ไฟล์ httpd.conf ใน server ของคุณด้วย หากไม่ได้เปิดมันจะไม่สามารถทำงานได้ครับ
