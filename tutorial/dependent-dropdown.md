การสร้าง Dropdown เพื่อสร้างตัวเลือก จังหวัด, อำเภอ, ตำบล
---------------------------
ในตัวอย่างนี้จะเป็นใช้ widget [Dependent Dropdown Widget](http://demos.krajee.com/widgets#depdrop) ของ krajee เพราะสามารถใช้งานได้ง่ายๆ แค่สร้าง action ตามที่ widget กำหนดโดยที่ไม่ต้องไปยุ่งกับ javascript มากมายเพราะ widget ทำไว้ให้หมดแล้ว เรามีหน้าที่ `config` ให้มันรู้ว่า action ใหนที่จะไปดึงข้อมูลอำเภอ action ใหนจะไปดึงข้อมูลตำบล

![dependent](/images/depdrop.png)

## หลักการทำงาน
![deropdown](/images/dropdown-flow.png)


- ขั้นตอนแรก จังหวัดเราจะทำการใช้ dropdownList แบบธรรมดาและคิวรีข้อมูลขึ้นมาเพื่อกำหนดค่าให้กับ dropdownList จังหวัดเพื่อแสดงผล
![deropdown](/images/province.png)

- เมื่อมีการคลิกเลือกจังหวัด ก็จะทำการส่งข้อมูลรหัสจังหวัด ที่เลือกไปที่ `actionGetAmphur()` เพื่อทำการค้นหาอำเภอ
 และจะส่งข้อมูลกลับมาในรูปแบบ `json` ไปให้ widget และนำไปแสดงผลที่ DropdownList อำเภอ
 ![deropdown](/images/amphur.png)
- เมื่อทำการเลือกอำเภอ ก็จะทำการส่งข้อมูลรหัสอำเภอที่เลือกไปที่ `actionGetDistrict()` เพื่อทำการค้นหาข้อมูลตำบล และส่งข้อมูลกลับให้ widget ในรูปแบบของ `json`เพื่อนำไปแสดงผลต่อที่ dropdownList ตำบล
![deropdown](/images/district.png)



## การติดตั้ง Widget
ในตัวอย่างนี้เราใช้ Widget [Dependent Dropdown Widget](Dependent Dropdown Widget) ซึ่งจะเป็นเพ็คเก็จที่รวมอยุ่ใน [Yii2 Widgets](http://demos.krajee.com/widgets) ซึ่งจะมี widget อื่นๆ อีกหลายตัว

รันคอมมานเพื่อทำการติดตั้ง
```
composer require kartik-v/yii2-widgets "*"
```
หรือเพิ่ม
```
"kartik-v/yii2-widgets": "*"
```
ที่แท็ค require ในไฟล์ composer.json จากนั้น `composer update`

## สร้าง Model
ทำการสร้าง gii Model ให้ครบทั้ง 3 ตารางคือ province, amphur, district
> ข้อมูล จังหวัด,อำเภอ,ตำบล [ดูได้ที่นี่](https://github.com/dimpled/Yii2-Learning-Source/blob/master/yii2-learning-source.sql)

## เรียกใช้งานใน Form
ทำการเรียกใช้งาน model ทั้งหมด เรียก ArrayHelper และ DepDrop
```php
<?php
use app\models\Province;
use app\models\Amphur;
use app\models\District;

use yii\helpers\ArrayHelper;

use kartik\widgets\DepDrop;
```

### สร้าง DropdownList จังหวัด
ในการเรียกข้อมูลเพื่อมาใช้กับ DropdownList โดยเรียกผ่าน model นั้นเราจะเป็นจะต้องใช้ `ArrayHelper` เพื่อสร้าง array ที่สามารถใช้กับ dropdownList ได้ โดยใช้ `ArrayHelper::map("model","ชื่อฟิวด์รหัส","ชื่อฟิวด์ที่เป็นข้อความ")`
```
ArrayHelper::map(Province::find()->all(),
'PROVINCE_ID',
'PROVINCE_NAME')
```
จากนั้นเราจะได้ Array กลับมาในรูปแบบ
```php
array(
  '28'=>'ขอนแก่น',
  '29'=>'มาหาสารคาม',
  .......
  )
```
เมื่อได้ข้อมูลแล้วเราก็เรียกใช้งานและทำการตั้งชื่อให้กับ DropdownList จังหวัด `ddl-province`

```php
<?= $form->field($model, 'province')->dropdownList(
            ArrayHelper::map(Province::find()->all(),
            'PROVINCE_ID',
            'PROVINCE_NAME'),
            [
                'id'=>'ddl-province',
                'prompt'=>'เลือกจังหวัด'
]); ?>
```

ซึ่งก็จะเห็นว่าเราสามารถเลือกจังหวัดได้แล้ว

![deropdown](/images/province-view.png)
