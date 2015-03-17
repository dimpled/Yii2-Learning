# รวมคำสั่ง Query ใน Model ที่ใช้งานบ่อยๆ

## Find()
เป็นคำสั่งค้นข้อมูลและส่งค่ากลับมาทีละแถว โดยใช้ primary key โดยสามารถใส่ค่าเข้าไปที่ parameter ของ `find()` ได้เลย หากค้าแล้วไม่เจอจะคืนค่าเป็น `null` คล้ายกับ `findByPk()` ใน yii1

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
เป็นคำสั่งที่ให้เราสามารถเพิ่มเงื่อนไขต่างๆ เข้าไปได้ สามารถใช้ร่วมกัน one() หรือ  all() ได้

ตัวอย่างการใช้งาน where() แบบต่างๆ
### Sample 1:
```php
$userid=1;
$model = User::find()
	->where('userid > :userid', [':userid' => $userid])
	->one();
```
### Sample 2:
```php
$model = User::find()
	->where(['reg_date' => $date, 'status' => 1])
	->one();
```
### Sample 3:
```php
$model = User::find()
	->where("reg_date > '2014-01-01' and status=1")
	->all();
```
### Sample 4:
เป็นการเพิ่มหลายๆ เงื่อนไขต่อกันสามารถระบุได้ว่าเงื่อนไขต่อไปเป็น and,หรือ or
```php
$model = User::find()
	->where('userid > :userid', [':userid' => $userid])
	->orWhere('primary_user = :primary_user', [':primary_user' => $primary_user])
	->andWhere('status = :status', [':status' => $status])
	->all();
```

หากดูในรูปแบบ sql จะได้แบบนี้
```sql
SELECT * FROM `tbl_user` WHERE ((userid > 1) OR (primary_user = 1)) AND (status = 1)
```

## orderBy()

### Sample 1:
```php
$model = User::find()
    ->where(['status' => 1])
    ->orderBy('userid')
    ->all();
```
### Sample 2:
```php
$model = User::find()
	->where(['status' => 0])
	->orderBy('userid')
	->one();
```
### Sample 3:
```php
	$model = User::find()
	->orderBy([
	       'usertype'=>SORT_ASC,
	       'username' => SORT_DESC,
		])
	->limit(10)
	->all();
```
ถ้าดูในรูปแบบ `sql` จะได้แบบนี้
```sql
SELECT * FROM `tbl_user` ORDER BY `usertype`, `username` DESC LIMIT 10
```

## count()
```php
	$model = User::find()
	->where(['status' => 0])
	->orderBy('userid')
	->count();

```

## asArray()
รับค่าข้อมูลกลับมาในรูปแบบของ Array ซึ่งปกติเราจะได้ในรูปแบบ Object
> ! หากใช้ asArray() เราจะไม่สามารถเรียกใช้งาน $model->username แบบนี้ได้ ต้องเป็น $model['username']  ต้องเลือกให้เหมาะกันงานที่จะนำมาใช้นะครับ
```php
$model = User::find()
	->asArray()
	->all();
```

```php
$model = User::find()
	->asArray()
	->one();

```
## indexBy()

```php
$model = User::find()
->indexBy('id')
->one();

```

## limit()

### Sample 1:
```php
$model = User::find()
->limit(10)
->all();
```
### Sample 2:
```php
$model = User::find()
->where('userid > 1 and isactive=1')
->limit(2)
->all();
```

## offset()
เป็นคำสั่งเรียกข้อมูล 5 แถว โดยเริ่มที่ แถวที่ 10
```php
$model = User::find()
		->limit(5)
		->offset(10)
		->all();
```
ดูในรูปแบบ `sql` จะได้แบบนี้
```sql
SELECT * FROM `tbl_user` LIMIT 5 OFFSET 10
```

## Limit With Pagination()
เป็นการใช้งาน model ร่วมกับ pagination สามารถตั้งค่า `defaultPateSize` ได้
```php
$query = Country::find();
$pagination = new Pagination([
			'defaultPageSize' => 5,
      'totalCount' => $query->count(),
        ]);

$countries = $query->orderBy('name')
	->offset($pagination->offset)
    ->limit($pagination->limit)
    ->all();


```
## LILE Condition()
เป็นการใช้งานคำสั่ง like ส่วนใหญ่เราก็ใช้ในกรณีค้นหาข้อมูล สามารถระบุ % หน้า % หลังได้

### Sqmple 1:
```php
$model = User::find()
		->where(['LIKE', 'username', 'admin'])
		->all();
//OR
$model = User::find()
		->where('username LIKE :query')
		->addParams([':query'=>'%admin%'])
		->all();
```
หากดูในรูปแบบ `sql` จะได้แบบนี้
```sql
SELECT * FROM `tbl_user` WHERE `username` LIKE '%admin%'
```

### Sqmple 2:
```php
$model = User::find()
		->where(['NOT LIKE', 'username', 'admin'])
		->all();
```
หากดูในรูปแบบ `sql` จะได้แบบนี้
```sql
SELECT * FROM `tbl_user` WHERE `username` NOT LIKE '%admin%'
```

## In Condition()
In เราสามารถระบบค่าเป็น array ได้เลย

### Sqmple 1:
```php
$model = User::find()
		->where([
			'userid' => [1001,1002,1003,1004,1005],
			])
		->all();
```
### Sqmple 2:
```php
$model = User::find()
		->where(['IN', 'userid', [1001,1002,1003,1004,1005]])
		->all();
```
หากดูในรูปแบบ `sql` จะได้แบบนี้
```sql
SELECT * FROM `tbl_user` WHERE `userid` IN (1001, 1002, 1003, 1004, 1005)
```
### Sqmple 3:
```php
$model = User::find()
		->where(['NOT IN', 'userid', [1001,1002,1003,1004,1005]])
		->all();

```
หากดูในรูปแบบ `sql` จะได้แบบนี้
```sql
SELECT * FROM `tbl_user` WHERE `userid` NOT IN (1001, 1002, 1003, 1004, 1005)
```

## between()

```php
$model = User::find()
		->select('username')
		->asArray()
		->where('userid between 1 and 5')
		->all();
```
หากดูในรูปแบบ `sql` จะได้แบบนี้
```sql
SELECT `username` FROM `tbl_user` WHERE userid between 1 and 5
```

## groupBy()
```php
$model = User::find()
		->groupBy('usertype')
		->all();
```
หากดูในรูปแบบ `sql` จะได้แบบนี้
```sql
SELECT * FROM `tbl_user` GROUP BY `usertype`
```

## having()

```php
$states=1;
$model = User::find()
		->groupBy('usertypee')
		->having('states >:states')
		->addParams([':states'=>$states])
		->all();
```
```sql
SELECT * FROM `tbl_user` GROUP BY `usertypee` HAVING states >1
```

## addParams()
เป็นการแทนค่าตัวแปรในเงือนไข where

### Sqmple 1:
```php
$usertype=1;
$model = User::find()
	->where('usertype = :usertype')
	->addParams([':usertype' => $usertype])
	->one();
```
### Sqmple 2:
```php
$usertype=1;
$status=0;
$model = User::find()
	->where('usertype = :usertype and status=:status')
	->addParams([':usertype' => $usertype])
	->addParams([':status' => $status])
// OR Multiple Assigns
// ->addParams([':usertype' => $usertype,':status' => $status])
	->one();

```

## Multiple Condition()
ในกรณีที่ต้องใช้หลายๆ เงื่อนไข

```php
$model = User::find()
		->where([
			'type' => 26,
			'status' => 1,
			'userid' => [1001,1002,1003,1004,1005],
			])
		->all();
```
ดูในรูปแบบ `sql` จะได้แบบนี้
```sql
SELECT * FROM `tbl_user` WHERE (`type`=26) AND (`status`=1) AND (`userid` IN (1001, 1002, 1003, 1004, 1005))

```

## Scope()

เป็นการสร้างเงื่อนไขที่เราใช้บ่อยๆ ให้สามารถเรียกใช้งานผ่าน functions ได้

ตามตัวอย่างด้านล่าง เป็นการสร้าง `scope` ชื่อ `olderThen` มีการรับค่า parameter 1 ตัว ชื่อ `$age`และมีการกำหนดค่า default ให้เท่ากับ 5 ในกรณีที่เรียกใช้งานแล้วไม่ได้ใส่ค่ามาด้วย `` paramerter มาด้วย `$age` จะเท่ากับ 5

### Sqmple 1:
```php
class User extends \yii\db\ActiveRecord
{
namespace app\models;
public static function olderThan($age = 5)
{
	$this->andWhere('userid > :age', [':age' => $age]);
							return $this;
}
}
// เรียกใช้งานและระบุค่า ซึ่งค่า `$age` จะเท่ากับ 7
$model = User::find()->olderThan(7)->all();

// เรียกใช้งานแบบไม่ระบุค่า ซึ่ง $age จะเท่ากับ 5
$model = User::find()->olderThan()->all();
```

### Sqmple 2:
ตัวอย่างนี้ไม่้ได้กำหนดค่า default ให้ จะต้องส่งค่ามาเสมอในตอนที่เรียกใช้งาน

```php

class User extends \yii\db\ActiveRecord
{
	// ...
	public static function active($status)
	{
			$this->andWhere('status = '.$status);
			return $this;
	}
}
// call the scope function
$model = User::find()
->active(1)
->all();

```


## findBySql()
เป็นการเรียกคำสั่ง sql โดยตรงผ่าน model

### Sqmple 1:
```php
$sql = 'SELECT * FROM tbl_user';
$model = User::findBySql($sql)->all();
```

### Sqmple 2:
```php
$sql = 'SELECT * FROM tbl_user';
$model = User::findBySql($sql)->all();
```
