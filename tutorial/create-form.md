การใช้งานฟอร์มแบบต่างๆ
-----------------------------------
ในตัวอย่างนี้ได้สร้างฟอร์มบันทึกข้อมูลพนักงาน ดูโครงสร้าง ตารางได้ที่

ทำการ gii model, gii crud ให้กับ ตาราง employee จากนั้นปรับแต่งฟอร์มตามรูปแบบต่อไปนี้

ไปที่ไฟล์ `/views/employee/_form.php`

## เปลี่ยนไปใช้งาน ActiveForm ของ Bootstrap

```php
use yii\widgets\ActiveForm;
```
เปลี่ยนเป็น

```php
use yii\bootstrap\ActiveForm;
```

## จัดฟอร์มให้เป็นแนวนอน
ปกติฟอร์มที่ crud มานั้นจะเป็นแนวตั้ง เราสามารถเปลี่ยนเป็นแนวนอนได้ดังนี้
```php
<?php $form = ActiveForm::begin(['layout' => 'horizontal']); ?>
```
จากเดิม

![vertical](/images/vertical.png)

จะได้แบบนี้

![vertical](/images/horizontal.png)

## การแบ่งให้คอลัมน์ให้ Field
> ตัวอย่างนี้จะไม่ได้ใช้ฟอร์มแบบแนวนอน (horizontal) ลองแบบแนวนอนแล้วยาก.. ฮา เอาแบบนี้ก่อนละกัน ถ้าทำได้จะมาบอกใหม่นะ

เปลี่ยนเป็นแบบเดิมก่อน..
```php
<?php $form = ActiveForm::begin(); ?>
```

เราสามารถแสดงผลข้อมูลของฟิวด์ให้อยู่ในแถวเดียวกันได้ ตัวอย่างจะใช้ คำนำหน้า,ชื่อ,นามกลุมมาไว้แถวเดียวกัน

![แถวเดียวกัน](/images/form-inline.png)

ก่อนอื่นต้องทำความเข้าใจ [Grid system](http://getbootstrap.com/css/#grid) ของ Bootstrap ก่อน

โดย Grid system จะมี 4 ขนาดคือ
- Phones  ใช้ class `col-xs-` คือหน้าจอที่มีขนาดกว้างน้อยกว่า 768 px ส่วนใหญ่ก็จะเป็นพวกมือถือ
- Tablets  ใช้ class `col-sm-` หน้าจอที่มีขนาดกว้างมากกว่า 768px ส่วนใหญ่ก็จะเป็นพวก tablets
- Desktops  ใช้ class `col-md-` หน้าจอที่มีขนาดกว้างมากกว่า 992px  ส่วนใหญ่ก็โน้ดบุ้คหรือเดสก์ท๊อป
- Large Desktops  ใช้ class `col-lg-`  หน้าจอที่มีขนาดกว้างมากกว่า 1200 px ส่วนใหญ่ก็จะเป็นเดสก์ท๊อปหรือที่ใช้จอใหญ่ๆ

> ในการใช้งานเราจะแบ่งคอลัมน์สูงสุดได้เพียง 12 คอลัมน์

### แบ่ง 2 คอลัมน์ สำหรับหน้าจอ  Mobile และ Tablets

แบ่งเป็น 2 คอลัมน์ และในตัวอย่างจะมี 2 เงื่อนไขคือ
-  `col-sm-` Tablets ถ้าความกว้างหน้าจอมากกว่า 768px และไม่เกิน 992px จะแสดงเป็น 3 คอลัมน์
- `col-md-` Desktops ถ้าความกว้างหน้าจอมากกว่า 992px ขึั้นไปก็จะแบ่งเป็น 3 คอลัมน์ แต่คอลัมน์จะกว้างกว่า

```html
<div class="row">
    <div class="col-sm-6 col-md-6">
       <?= $form->field($model, 'position_id')->textInput() ?>
    </div>
    <div class="col-sm-6 col-md-6">
       <?= $form->field($model, 'mobile_phone')->textInput(['maxlength' => 20]) ?>
    </div>
</div>
```

### แบ่ง 3 คอลัมน์ สำหรับหน้าจอ Mobile, Tablets และ Desktop

เราสามารถกำหนดขนาดหน้าจอได้มากกว่า 1 อัน ตัวอย่างนี้กำหนด 3 ขนาด

```html
<div class="row">
    <div class="col-xs-6 col-sm-4 col-md-4">
       <?= $form->field($model, 'email')->textInput(['maxlength' => 100]) ?>
    </div>
    <div class="col-xs-6 col-sm-4 col-md-4">
       <?= $form->field($model, 'birthday')->textInput() ?>
    </div>
    <div class="col-xs-12 col-sm-4 col-md-4">
        <?= $form->field($model, 'salary')->textInput() ?>
    </div>
</div>
```

ตัวอย่าง

`col-xs-`  Browser มันเช็คว่าเป็นหน้าจอที่ขนาดน้อยกว่า 768px จะแบ่งเป็น 2 คอลัมน์เท่าๆ กันด้านละ 6 แต่คอลัมน์ที่ 3 ตั้งค่าไว้ที่ 12 จะตกไปอยู่แถวถัดไป
แต่ถ้าน้อยกว่า

![3 column](/images/3-1.png)

`col-sm-` browser มันเช็คว่าเป็นหน้าจอที่ขนาดมากกว่า 768px จะแบ่งเป็น 3 คอลัมน์เท่าๆ กัน

![3 column](/images/3.png)

 `col-md-` browser มันเช็คว่าเป็นหน้าจอที่ขนาดมากกว่า 992px จะแบ่งเป็น 3 คอลัมน์เท่าๆ กัน

 ![3 column](/images/3-full.png)


 ## การใช้งาน DateField

ต้องทำการติดตั้ง yii2-jui ก่อน

รันคำสั่ง

```
composer require --prefer-dist yiisoft/yii2-jui
```
หรือ

```
"yiisoft/yii2-jui": "~2.0.0"
```

เพิ่มที่ไฟล์ `composer.json` แล้วรันคำสั่งคอมมาน `composer update`
- `dateFormat` จัด format date ที่จะทำการบันทึก
- `changeMonth` ให้มีตัวเลือกเดือน
- `changeYear` ให้มีตัวเลือกปี


 ```php
 <?= $form->field($model, 'birthday')->widget(DatePicker::classname(), [
        'language' => 'th',
        'dateFormat' => 'yyyy-MM-dd',
        'clientOptions'=>[
          'changeMonth'=>true,
          'changeYear'=>true,
        ],
        'options'=>['class'=>'form-control']
      ]) ?>

```
