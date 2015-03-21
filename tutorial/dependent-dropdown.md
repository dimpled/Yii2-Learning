การใช้งาน Dropdown เพื่อสร้างตัวเลือก จังหวัด, อำเภอ, ตำบล
---------------------------
ในตัวอย่างนี้จะเป็นใช้ widget [Dependent Dropdown Widget](http://demos.krajee.com/widgets#depdrop) ของ krajee เพราะสามารถใช้งานได้ง่ายๆ เพียงแค่สร้าง action ตามที่ widget กำหนดโดยที่ไม่ต้องไปยุ่งกับ javascript มากมายเพราะ widget ทำไว้ให้หมดแล้ว เรามีหน้าที่ `config` ให้มันรู้ว่า action ใหนที่จะไปดึงข้อมูลอำเภอ action ใหนจะไปดึงข้อมูลตำบล

![dependent](/images/depdrop.png)

## หลักการทำงาน
เราจะทำการสร้าง DropdownList ไวัทั้งหมด 3 ตัว ตัวแรก จังหวัดจะเป็น DropdownList ธรรมดา และดึงข้อมูลจังหวัดมาแสดง ส่วนอีก 2 ตัวที่เลือกจะทำการเรียกข้อมูลผ่าน request ajax เมื่อมีการคลิกเลือกอำเภอและตำบลตามลำดับ 

![deropdown](/images/dropdown-flow.png)


- ขั้นตอนแรก ตัวเลือกจังหวัด เราจะทำการใช้ dropdownList แบบธรรมดาและคิวรีข้อมูลขึ้นมาเพื่อกำหนดค่าให้กับ dropdownList จังหวัดเพื่อแสดงผล
![deropdown](/images/province.png)

- เมื่อมีการคลิกเลือกจังหวัด ก็จะทำการส่งข้อมูลรหัสจังหวัด ที่เลือกไปที่ `actionGetAmphur()` เพื่อทำการค้นหาอำเภอ
 และจะส่งข้อมูลกลับมาในรูปแบบ `json` ไปให้ widget และนำไปแสดงผลที่ DropdownList อำเภอ
 ![deropdown](/images/amphur.png)
- เมื่อทำการเลือกอำเภอ ก็จะทำการส่งข้อมูลรหัสอำเภอที่เลือกไปที่ `actionGetDistrict()` เพื่อทำการค้นหาข้อมูลตำบล และส่งข้อมูลกลับให้ widget ในรูปแบบของ `json`เพื่อนำไปแสดงผลต่อที่ dropdownList ตำบล
![deropdown](/images/district.png)

> ตัว widgit สามารถสร้างตัวเลือกสับย่อยๆ เข้าไปอีกได้ อาจจะมากกว่า 3 ก็ได้ ก็แล้วแต่กรณีที่เราจะนำไปใช้ เช่น ข้อมูลหมวดหมู่ประเภทสินค้า

## การติดตั้ง Widget
ในตัวอย่างนี้เราใช้ Widget [Dependent Dropdown Widget](Dependent Dropdown Widget) ซึ่งจะเป็นเพ็คเก็จที่รวมอยุ่ใน [Yii2 Widgets](http://demos.krajee.com/widgets) ซึ่งจะมี widget อื่นๆ อีกหลายตัว
> widget นี้ใช้งานร่วมกับ  [yiisoft/yii2-bootstrap](https://github.com/yiisoft/yii2/tree/master/extensions/bootstrap) แต่ส่วนใหญ่ก็ติดตั้งมาพร้อมอยู่แล้ว

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

## สร้าง DropdownList จังหวัด
ในการเรียกข้อมูลเพื่อมาใช้กับ DropdownList โดยเรียกผ่าน model นั้นเราจะเป็นจะต้องใช้ `ArrayHelper` เพื่อสร้าง array ที่สามารถใช้กับ dropdownList ได้ โดยใช้ `ArrayHelper::map("model","ชื่อฟิวด์รหัส","ชื่อฟิวด์ที่เป็นข้อความ")`
```php
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
เมื่อได้ข้อมูลแล้วเราก็เรียกใช้งานและทำการตั้งชื่อให้กับ DropdownList จังหวัดชื่อ `ddl-province`

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

> ตอนนี้เราตั้ง id dropdownlist `ddl-province`

## สร้าง DropdownList อำเภอ
เราจะใช้ widget `DepDrop` เพื่อทำการสร้าง dropdownlist และกำหนดชื่อเป็น `ddl-amphur`  ซึ่งจะเป็นอำเภอว่างๆ ไว้เพื่อรอให้มีการส่งข้อมูลมา ซึ่งตัว dropdownList อำเภอจะมีการกำหนดค่า 3 ตัวคือ
- `data` ตรงนี้เอาไว้ใช้รับมูลมาแสดงในกรณี `update` เพื่อให้แสดงค่าว่าเราได้เลือกอำเภอใหนแต่ตอนนี้ใส่เป็น array ว่างๆ ไว้ก่อน
- `depends` เป็นการระบุชื่อ dropdownlist จังหวัดเพื่อจับ event เมื่อมีการคลิกเลือกจังหวัด
- `url` เป็นการระบุชื่อ action ที่จะให้ dropdownlist ไปเรียกข้อมูลอำเภอ

โดยกำหนดค่าดังนี้

```php
<?= $form->field($model, 'amphur')->widget(DepDrop::classname(), [
            'options'=>['id'=>'ddl-amphur'],
            'data'=> [],
            'pluginOptions'=>[
                'depends'=>['ddl-province'],
                'placeholder'=>'เลือกอำเภอ...',
                'url'=>Url::to(['/employee/get-amphur'])
            ]
        ]); ?>
```


## สร้าง DropdownList ตำบล
การกำหนดค่าจะคล้ายกับ อำเภอ แต่ต่างกันที่ `depends` จะมีการกำหนดค่าชื่อไว้สองตัวคือ ชือ dropdownlist จังหวัดและ dropdownlist อำเภอ
```php
<?= $form->field($model, 'district')->widget(DepDrop::classname(), [
           'data' =>[],
           'pluginOptions'=>[
               'depends'=>['ddl-province', 'ddl-amphur'],
               'placeholder'=>'เลือกตำบล...',
               'url'=>Url::to(['/employee/get-district'])
           ]
]); ?>
```
เราจะได้ form แบบนี้

![deropdown](/images/depdrop-all.png)

แต่ตอนนี้จะยังใช้งานไม่ได้เพราะเรายังไม่ได้สร้าง action เพื่อรองรับการค้นข้อมูลและส่งข้อมูลให้กับ dropdownlist

## สร้าง Action เพื่อรองรับการคลิกเลือกจังหวัดและอำเภอ
เราจะทำการสร้าง action ไว้ที่ controllers/EmployeeController.php
action แรกจะเป็น actionGetAmphur() ตรงนี้จะรองรับการคลิกเลือกจังหวัดและส่งค่าอำเภอกลับไปให้กับ dropdownlist อำเภอเป็น json

```php
public function actionGetAmphur() {
     $out = [];
     if (isset($_POST['depdrop_parents'])) {
         $parents = $_POST['depdrop_parents'];
         if ($parents != null) {
             $province_id = $parents[0];
             $out = $this->getAmphur($province_id);
             echo Json::encode(['output'=>$out, 'selected'=>'']);
             return;
         }
     }
     echo Json::encode(['output'=>'', 'selected'=>'']);
 }
```
action ที่สองจะเป็น actionGetDistrict() ตรงนี้จะรองรับการคลิกเลือกอำเภอและ ส่งค่าตำบลกลับไปให้กับ dropdownlist ตำบลเป็น json

```php
 public function actionGetDistrict() {
     $out = [];
     if (isset($_POST['depdrop_parents'])) {
         $ids = $_POST['depdrop_parents'];
         $province_id = empty($ids[0]) ? null : $ids[0];
         $amphur_id = empty($ids[1]) ? null : $ids[1];
         if ($province_id != null) {
            $data = $this->getDistrict($amphur_id);
            echo Json::encode(['output'=>$data, 'selected'=>'']);
            return;
         }
     }
     echo Json::encode(['output'=>'', 'selected'=>'']);
 }
```

action actionGetAmphur() ข้างใน function จะมีการเรียกใช้งาน getAmphur($id) และส่งค่ารหัส จังหวัดมาให้ เพื่อนำไปค้นที่ model Amphur
```php
 protected function getAmphur($id){
     $datas = Amphur::find()->where(['PROVINCE_ID'=>$id])->all();
     return $this->MapData($datas,'AMPHUR_ID','AMPHUR_NAME');
 }
 ```

 action actionGetDistrict() ข้างใน function จะมีการเรียกใช้งาน getDistrict($id) และส่งค่ารหัส อำเภอมาให้ เพื่อนำไปค้นที่ model District
 ```php
 protected function getDistrict($id){
     $datas = District::find()->where(['AMPHUR_ID'=>$id])->all();
     return $this->MapData($datas,'DISTRICT_ID','DISTRICT_NAME');
 }
 ```

 และ function `getAmphur()` และ `getDistrict()` จะทำการเรียก  `MapData` เพื่อทำการนำค่าที่ได้สร้าง array ในรูปแบบ `['id'=>'','name'=>'']` เพราะตัว widget DepDrop ได้ระบบให้สร้างตาม format นี้
 ```php
 protected function MapData($datas,$fieldId,$fieldName){
     $obj = [];
     foreach ($datas as $key => $value) {
         array_push($obj, ['id'=>$value->{$fieldId},'name'=>$value->{$fieldName}]);
     }
     return $obj;
 }
 ```

## ทดลองใช้งาน
เลือกจังหวัด
![select-province](/images/select-province.png)

เลือกอำเภอ

![select-province](/images/select-amphur.png)

เลือกตำบล

![select-province](/images/select-district.png)

## กรณี Update

ในกรณีอัพเดทเราจะเป็นจะต้องนำค่า อำเภอและตำบลที่บันทึกไว้ไปค้นเพื่อนำมาแสดงผลว่าเคยเลือกอะไรไปแล้ว

ไปที่ actionUpdate() ทำการเรียกฟังชั่น getAmphur(), getDistrict() เพื่อดึงข้อมูลไปแสดงที่ dropdownlist อำเภอและตำบลให้มันเลือกที่รายการที่เราเคยเลือกไว้ก่อนหน้านี้

```php

  public function actionUpdate($id)
  {
      $model          = $this->findModel($id);

      $amphur         = ArrayHelper::map($this->getAmphur($model->province),'id','name');
      $district       = ArrayHelper::map($this->getDistrict($model->amphur),'id','name');

    //..........

    return $this->render('update', [
           'model' => $model,
           'amphur'=> $amphur,
           'district' =>$district

    ]);
  }

```

และที่ ไฟล์ views/employee/update.php อย่างลืมส่งค่าไปให้ฟอร์มด้วย
```php
<?= $this->render('_form', [
        'model' => $model,
         'amphur'=> $amphur,
         'district' =>$district
  ]) ?>
```

จากนั้นเปลี่ยนค่า `data` ใน dropdownlist อำเภอและตำบลเป็นค่าที่ส่งมา
```php

<?= $form->field($model, 'amphur')->widget(DepDrop::classname(), [
    'options'=>['id'=>'ddl-amphur'],
    'data'=> $amphur, //<---------
    'pluginOptions'=>[
        'depends'=>['ddl-province'],
        'placeholder'=>'เลือกอำเภอ...',
        'url'=>Url::to(['/employee/get-amphur'])
    ]
]); ?>


<?= $form->field($model, 'district')->widget(DepDrop::classname(), [
    'data' =>$district, //<---------
    'pluginOptions'=>[
        'depends'=>['ddl-province', 'ddl-amphur'],
        'placeholder'=>'เลือกตำบล...',
        'url'=>Url::to(['/employee/get-district'])
    ]
]); ?>
```

อีกอย่างอย่างลืมสร้างตัวแปร array ว่างๆ ให้กับ /views/employee/create.php เพื่อในตอน create จะได้ไม่ error ตัวแปรไม่ได้ถูกสร้าง

```php
<?= $this->render('_form', [
        'model' => $model,
        'amphur'=> [],
        'district' =>[],
    ]) ?>

```

ลองใช้กันดูนะครับ
ดาวโหลด source code [ ได้ทีนี่](https://github.com/dimpled/Yii2-Learning-Source/releases)
