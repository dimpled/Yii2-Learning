# รวมคำสั่ง Query ใน Model ที่ใช้งานบ่อยๆ

## Find()
เป็นคำสั่งค้นข้อมูลและส่งค่ากับมาทีละแถว โดยใช้ primary key โดยสามารถใส่ค่าเข้าไปที่ parameter ของ `find()` ได้เลย หากค้าแล้วไม่เจอจะคืนค่าเป็น `null` คล้ายกับ `findByPk()` ใน yii1

ตัวอย่างการใช้งาน
```php
$model = User::find(45);
if($model){
	echo $model->username;
	echo $model->status;
}
```

## All()
เป็นคำสั่งค้นข้อมูลโดยไม่ได้ระบุเงื่อนไขและส่งค่ากับมาทีละหลายๆแถว โดยใช้ `method` all()
> ในการใช้งานจริงๆ เราสามารถใช้งานร่วมกับ method อื่นได้ เช่น where(),order() การใช้งานดูในตัวอย่างถัดๆ ไป

ตัวอย่างการใช้งาน
```php
$model = User::find()->all();

```
## One()
เป็นคำสั่งค้นข้อมูลโดยไม่ได้ระบุเงื่อนไขและส่งค่ากับมาทีละแถว โดยใช้ `method` one()
> ในการใช้งานจริงๆ เราสามารถใช้งานร่วมกับ method อื่นได้ เช่น where(),order() การใช้งานดูในตัวอย่างถัดๆ ไป

ตัวอย่างการใช้งาน
```php
$model = User::find()->one();

```

## where()
```php
$model = User::find()->one();

```
## orderBy()
```php
$model = User::find()->one();

```
## count()
```php
$model = User::find()->one();

```
## asArray()
```php
$model = User::find()->one();

```
## indexBy()
```php
$model = User::find()->one();

```
## limit()
```php
$model = User::find()->one();

```
## offset()
```php
$model = User::find()->one();

```
## Limit With Pagination()
```php
$model = User::find()->one();

```
## LILE Condition()
```php
$model = User::find()->one();

```
## In Condition()
```php
$model = User::find()->one();

```
## between()
```php
$model = User::find()->one();

```
## groupBy()
```php
$model = User::find()->one();

```
## having()
```php
$model = User::find()->one();

```
## addParams()
```php
$model = User::find()->one();

```
## Multiple Condition()
```php
$model = User::find()->one();

```
## Scope()
```php
$model = User::find()->one();

```
## findBySql()
```php
$model = User::find()->one();

```
