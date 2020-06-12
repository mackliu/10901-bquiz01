# 題組一解題步驟
#### 解題參考影片
**https://www.youtube.com/watch?v=lCONL7m-TZY&list=PLL26U2k-yzXvcwhiaTfgpwdLzbc_txCd5**

## 步驟一：將素材目錄複製到崗位目錄下，確認素材內容與抽題題號一致
監評長按下倒數計時後，可以先把桌面上素材目錄中的題目素材複製一份到自己的工作目錄下，這時要確認自己複製的題目和抽到的題目是一致的，之後都在工作目錄下來取用相關的素材，這樣比較不容易出錯；在安裝軟體的準備時間裏，也要確認一下電腦桌面中是否有包含了素材這個目錄，並且四個題組的素材都在其中。

---

## 步驟二：將版型檔案及相關素材複製到網站根目錄下，並進行相應的更名及整理
  1. 開立./css, ./js, ./img, ./icon等常用目錄以利檔案分類及管理
  2. 將素材檔中的.css, .js, 及icon圖檔複製到相應的目錄下
  3. 更改版型素材的相關檔名，以符合解題的需要
      * 01P01.html => login.php
      * 01P02.html => index.php
      * 01P03.html => admin.php
      * 01P04.html => news.php
  4. 更改版型素材的相關連結及匯入檔內容
      * 修改 `index.php`,`admin.php` 中 `<link>` 及 `<script>` 中的連結路徑，指向正確的位置
      * 修改 `./css/css.css` 中的圖片 `url` ，指向根目錄下的 `../icon` 目錄
  5. 開啟 `xampp` 及 `apache` 伺服器，使用 `localhost` 或 `127.0.0.1` 檢視網頁是否正確顯示，css 的載入是否正確

---

## 步驟三：進行前後台的檔案整理及切版，分離出共用的區塊或功能。
  1. 建立 `front`及 `backend` 兩個目錄，一個代表前台的相關檔案，一個代表後台的相關檔案，前後台共用的元件則先放在根目錄下，或另開一個 `comm` 目錄用來存放共用的元件
  2. 從`index.php`及`admin.php`中分離出中間需要變動的區塊，採用`include`的方式來動態載入主要的內容區
  3. 前台的 `login.php` 及 `news.php` 去除和 `index.php` 相同的部份，只留下中間區塊即可，並將兩個檔案移到 `./front` 目錄下
  4. 前台的 `index.php` 控出的中間區塊成為獨立的 `home.php` 檔案，並搬移到 `./front/` 目錄下
  5. 後台的 `admin.php` 則挖出中間的區塊成為獨立的元件，並搬移到`backend`目錄下，先建立一個檔案名為`title.php`，之後會成為九個後台功能的基礎版型檔案。
  6. 使用 `include` 指令來重新組合 `index.php` 及 `admin.php` 頁面，並加上判斷式來確保要組合的檔案是存在的。
  7. 以 `get` 的方式來傳遞各頁面要組合的元件內容，比如 `do=login` 表示要看到的是登入頁面，因此在前台的 `include` 中可以併入 `login.php` 來呈現。

**index.php**
```php
	<?php
		$do=(!empty($_GET['do']))?$_GET['do']:'main';
		$file='front/'.$do.".php";
		if(file_exists($file)){
			include $file;
		}else{
			include 'front/main.php';
		}
	?>
```
**admin.php**
```php
    <?php
	    $do=(!empty($_GET['do']))?$_GET['do']:'title';
	    $file='backend/'.$do.".php";
	    if(file_exists($file)){
	    	include $file;
	    }else{
	    	include 'backend/title.php';
	    }
	?>
```
  8. 在 `./front` 目錄中，將 `login.php`, `news.php`, `home.php` 中的 `<marquee>...</marquee>` 也獨立成為一個元件，並放在 `./front/` 目錄下
  9. 修改 `admin.php` 左方選單中的連結內容由 `href="?do=admin&redo=title"` 改成 `href="?do=title"`，並確認連結可以看到對應的功能內容


```
note:
news.php 及 home.php 下方的<script></script>是用來做為最新消息彈出視窗用的，
因此在切割檔案時，要記得連<script>的部份一起切出去
```

---

## 步驟四：建立資料庫連線檔及常用函式。
  1. 建立 `base.php` 檔，用來放共用的設定及函式。
  2. 採用類別方式來包裝整個db的連線及資料表的存取函式

```php

class DB{

//類別內容

}

```
  3. 設定好PDO的連線參數 `$pdo=new PDO()`

```php

    private $dsn="mysql:host=localhost;charset=utf8;dbname=db88";
    private $root="root";
    private $password="";
    private $pdo;

    //設定建構式
    public function __construct($table){

        //將建立物件時代入的資料表名稱代入類別中的屬性table
        $this->table=$table

        //建立pdo的連線資訊，並將pdo連線指定給類別內的屬性pdo
        $this->pdo=new PDO($this->dsn,$this->root,$this->password); 
    }

```
  4. 建立全域變數或是共用函式
      * find(...\$arg) - 尋找特定條件的單筆資料或第一筆資料
      * all(\$arg) - 取得資料表的全部資料或是特定條件的全部資料
      * count(...\$arg) - 計算符合條件的資料筆數
      * save(\$arg) - 新增或更新單筆資料
      * del(\$arg) - 刪除特定條件的全部資料
      * q(\$sql) - 簡化 \$pdo->query(\$sql)->fetchAll() 的使用;
      * to(\$url) - 簡化 header("location:xxxxxx") 的使用;
```php
class DB{
    //......

  public function all(...$arg){
    //...... 
  }
  public function find($arg){
    //......
  }
  public function count(...$arg){
    //......
  }
  public function save($arg){
    //......
  }
  public function del($arg){
    //......
  }
  public function q($sql){
    //......
  }

}


```
  5. 啟用session `session_start()`
  6. 做好以上工作後，可以先建一張簡單的資料表，把資料庫連線及所有自訂函式功能先測試一次，以確保後續使用不會有問題。

---

## 步驟五：建立資料表及預設資料。
每個題組依狀況不同，在這一步有不同的做法，視自己對題目的熟悉程度來做應變，可以一次把全部資料表建完，也可以視解題的進度來逐步建立或修改資料表。
這裏我們採用的做法是利用phpmyadmin的複製資料表功能，快速的複製五張欄位相同的資料表（title,ad,mvim,image,news)，
五張類似的資料表並不是所有的欄位都會用得上，我們只是取巧來節省建資料表的時間。

1. 依序建立後台功能需要的九張資料表:
  * title

    | name |  type  |  pk | default | A_I |   note   |
    |:----:|:------:|:---:|:-------:|:---:|:--------:|
    |id    |int(5)  |yes  |         |yes  | 流水號    |
    |img   |text    |     |         |     | 檔名/路徑 |
    |text  |text    |     |         |     | 文字      |
    |sh    |int(1)  |     |   0     |     | 顯示      |
    
  * ad

    | name |  type  |  pk | default | A_I |   note   |
    |:----:|:------:|:---:|:-------:|:---:|:--------:|
    |id    |int(5)  |yes  |         |yes  | 流水號    |
    |img   |text    |     |         |     | 檔名/路徑 |
    |text  |text    |     |         |     | 文字     |
    |sh    |int(1)  |     |   1     |     | 顯示     |

  * mvim
  
    | name |  type  |  pk | default | A_I |   note   |
    |:----:|:------:|:---:|:-------:|:---:|:--------:|
    |id    |int(5)  |yes  |         |yes  | 流水號    |
    |img   |text    |     |         |     | 檔名/路徑 |
    |text  |text    |     |         |     | 文字     |
    |sh    |int(1)  |     |   1     |     | 顯示     |

  * image

    | name |  type  |  pk | default | A_I |   note   |
    |:----:|:------:|:---:|:-------:|:---:|:--------:|
    |id    |int(5)  |yes  |         |yes  | 流水號    |
    |img   |text    |     |         |     | 檔名/路徑 |
    |text  |text    |     |         |     | 文字      |
    |sh    |int(1)  |     |   1     |     | 顯示      |

  * total

    | name |  type  |  pk | default | A_I |   note   |
    |:----:|:------:|:---:|:-------:|:---:|:--------:|
    |id    |int(5)  |yes  |         |yes  |流水號     |
    |total |int(5)  |     |         |     |訪客數     |

  * bottom

    | name |  type  |  pk | default | A_I |   note   |
    |:----:|:------:|:---:|:-------:|:---:|:--------:|
    |id    |int(5)  |yes  |         |yes  |流水號     |
    |bottom|text    |     |         |     |頁尾版權   |

  * news

    | name |  type  |  pk | default | A_I |   note   |
    |:----:|:------:|:---:|:-------:|:---:|:--------:|
    |id    |int(5)  |yes  |         |yes  | 流水號    |
    |img   |text    |     |         |     | 檔名/路徑 |
    |text  |text    |     |         |     | 文字      |
    |sh    |int(1)  |     |   1     |     | 顯示      |

  * admin

    | name |  type  |  pk | default | A_I |   note   |
    |:----:|:------:|:---:|:-------:|:---:|:--------:|
    |id    |int(5)  |yes  |         |yes  | 流水號    |
    |acc   |text    |     |         |     | 帳  號    |
    |pw    |text    |     |         |     | 密  碼    |

  * menu

    | name |  type  |  pk | default | A_I |   note   |
    |:----:|:------:|:---:|:-------:|:---:|:--------:|
    |id    |int(5)  |yes  |         |yes  | 流水號    |
    |name  |text    |     |         |     | 文字      |
    |href  |text    |     |         |     | 連結      |
    |parent|int(5)  |     |         |     | 主選單id  |
    |sh    |int(1)  |     |  1      |     | 顯示      |

2. total,bottom,admin這三張表可以先直接手動塞一筆資料進去，如果對資料夠熟悉，也可以每張表都先塞資料進去，這樣在後續製作功能時，可以更快看到成果
3. 為了解題順利，可以把資料表中的一些欄位設為可接受空值的狀況，這樣即使未設定內容，也能正常新增或更改資料，不過這個做法只是為了先求解題完成而做的取巧，實務上應該根據需求及功能來決定欄位是否可以接受空值，並在程式端檢查來源資料是否為空值

---

## 步驟六：製作訪客計數器及頁尾版權文字
由於第一題的前置作業較多，因此建議先把訪客計數器先完成，確認自訂函式及資料庫的存取正常，一來是先看到有個功能完成會比較心安，二來是確認一下前置作業的自訂函式部份有沒有問題。

1. 先整理後台的進站總人數管理功能的頁面，調整HTML的部份符合題意要求的單欄資料內容
2. 確認可從資料表 `total` 中讀取到訪客計數資料
3. 建立 `./api/total.php` 並將資料更新的語法寫入
4. 在 `base.php` 中寫入判斷訪客是否是首次進站，如為首次進站則建立 `session` 並更新資料表中的進站人數。
5. 要注意的是原本的版型檔案中使用 `iframe` 的方式來傳遞表單資料，因此 `<form>` 標籤中會有 `target="back"` 的設定，但我們不打算使用iframe，因此要拿掉 `index.php`及`admin.php`中的 `<iframe>`，並將原本的 `<form>` 標籤內容略做修改：

```html
form method="post" target="back" action="?do=tii"
  改成
form method="post" action="./api/edit_info.php"
```
6. 在 `index.php`及`admin.php`的進站總人數位置，直接從資料表讀取資料來顯示
7. 完成進站人數的統計後，可以按照一樣的流程來製作頁尾版權文字，直接從資料表中取得資料即可
8. 完成頁尾版權的後台功能後，在 `index.php`及`admin.php` 中加入讀取頁尾版權資料的程式碼，如此一來就可以看到頁尾版權資料的內容

---

## 步驟七：製作後台網站標題管理功能
除了 `total`及`bottom`，其他七項後台的功能版面都很像，我們先以"網站標題管理"這個功能來做示範，先完成這個功能後，相同的程式碼可以快速套用到其它功能去；
我們需要先把素材提供的html整理一下，以符合我們解題的需要，另外，素材並沒有附上**新增資料**及**更新圖片**的表單格式，因此這部份的HTML碼要我們自己來撰寫：
1. 先移除 `admin.php` 中的 `iframe` 及修改 `<form>` 標籤的內容
2. 依照功能的要求修改列表欄位的HTML碼
3. 建立一個資料表名專用的變數來取代手動變更資料表名
4. 撰寫列出資料表內容的語法，依照各功能的要求，每個功能的語法可能略有不同
5. 將彈出視窗的js函式 `op` 套用到**更新圖片**按鈕中
6. 在彈出視窗的語法中加入必要的網址參數，如 `id`
7. 建立一個 `modal` 資料夾來存放使用ajax載入的彈出視窗內容
8. 指定新增/更新/編輯三個不同功能對應的api檔案及路徑名，如果需要帶入參數的也需一併填寫。
   * 需注意功能中是否需要判斷檔案上傳的動作
   * 需注意資料表名及函式的參數引用是否適合
   * 和檔案相關的操作要注意路徑或是檔案覆蓋的問題
   * 瀏灠器不會主動更新同檔名的圖片，因此在更新圖片的功能中，我們採用更新資料表內容的方式來強迫瀏灠器去更新圖片
   * 確認 `./img/` 目錄存在，上傳的檔案才有地方放

```
note:
實際解題時如果一個功能一個功能照順序做，那幾乎不太可能在時間完成，
因此這邊只是示範單一功能的完整開發過程；
實際應檢時，會同時考量多個類似功能的情形，並在完成一個模組後，快速複製套用；
比如我們延用網址參數$do來當成table的變數，在完成標題管理的HTML程式碼後，
可以先把修改好的HTML碼快速複製到類似的ad,mvim,image,news,admin,menu去，
然後修改欄位到符合題目要求，接著變更變數$useTable的值，
這樣就可以快速完成前端頁面的HTML碼處理，然後再接著寫API，
API的撰寫也會考量是否可以同時適用多個功能。
```

---

## 步驟八：套用後台的title頁面設計到其他後台功能中
完成網站標題圖片管理的功能後，其它七個後台項目的畫面及功能都差不多，所以可以快速的複製相關的內容過去，由於我們有先設了一個資料表的變數在，因此只需要變更這個變數的值，就可以確保其它功能或網址對應到的資料表名稱都是正確的。

相應的新增及更新圖片的表單檔案也是如法泡製的快速複製就可以了，如果不想檔案數太多，則使用switch case的方式來集中內容在一個檔案中也可以。

這個步驟中比較花時間的地方在於調整欄位到符合題目要求，建議要多練幾次，把各功能的欄位熟悉一下，速度才會快。

除了 `./backend/`目錄中的檔案，也要記得同時修改　`./modal/` 目錄中對應的彈出視窗

在表單的傳送目標 `action` 屬性，根據不同的功能，要記得修改不同的目標，如果是圖片上傳的表單，記得加上編碼宣告 `enctype="multipart/form-data"` 。

由於每個項目除了欄位外，都還有一些小地方有不同，因此在修改時要特別細心：
  * 除了標題圖片外，其他的項目顯示都是可多選的
  * 最新消息管理是使用 `textarea` 而不是 `input` 來顯示內容
  * 動態文字廣告及最新消息的文字欄位大小可以使用行內樣式直接調整即可
  * 校園映像圖片的顯示大小題目有要求，前台150x103，後台100x68，一樣使用行內樣式來設定即可

最後，記得檢查一下每個功能中，使用彈出視窗功能時，有沒有帶入對應的值或參數。

---

## 步驟九：修改api中的 add / edit 中和資料表有關的程式碼

在先前的**網站標題圖片**的項目中，我們讓資料的欄位名稱都相同，所以在API中，我們使用$data陣列在存放資料時，這個陣列的內容和資料表的欄位必須是一致的，因此才能使用save()這個函式去做新增和更新的動作。

但是在管理者帳號和選單資料表中，資料表的欄位和我們在API中使用的不同，因此會造成在新增及更新時，函式無法送出符合資料表欄位的語法，因此管理者帳號和選單資料表這兩個功能無法新增及編輯。

在原本的edit程式中，我們處理資料是否顯示是採用單選的方式來處理，但是除了**網站標題圖片**外的功能大多都是多選的，因此這部份的程式也需要做修改。

針對以上提到的有差異的地方，我們採用 switch...case 的方式，讓不同資料表對應的功能可以在api的地方做出差異。

```php
        //依據不同的資料表來做不同的動作
        switch($table){
            case "title":
                //將欄位內容更新成表單傳遞過來的內容
                $data['text']=$_POST['text'][$key];
                $data['sh']=($id==$_POST['sh'])?1:0;
            break;
            case "admin":
                $data['acc']=$_POST['acc'][$key];
                $data['pw']=$_POST['pw'][$key];                
            break;
            case "menu":
                $data['text']=$_POST['text'][$key];
                $data['href']=$_POST['href'][$key];
                $data['sh']=(in_array($id,$_POST['sh']))?1:0;                  
            break;
            default:
                //將欄位內容更新成表單傳遞過來的內容
                $data['text']=$_POST['text'][$key];
                $data['sh']=(in_array($id,$_POST['sh']))?1:0;
        }

```

我們在API這邊設計的主要考量是希望功能類似的就儘量套用同樣的程式碼，因此我們透過$table這個變數，讓同一支API程式可以自動去判斷要對那一張資表進行操作，這樣才可以大幅度的減少需要撰寫的程式碼，同時透過變數的應用，也可以減低出錯率。

---

## 步驟十：製作編輯次選單功能
次選單功能是本題組中較為複雜，但說明卻相對模糊的功能，這裏我們採用較直覺的做法來解題，依照題目給出的參考圖來看，題目希望次選單的新增/修改/刪除都在彈出視窗中完成。

由於一個畫面的表單中要同時具有增改刪查的功能，因此我們無法延用先前製作的API來處理次選單的功能，所以次選單的API單獨一支程式來處理，因此我們在 `./backend/menu.php` 的彈出視窗的按鈕參數上採用指定路徑檔名的方式來處理，而不是和先前幾個功能一樣採用帶入資料表變數的方式。

在API的部份，我們透過表單中的name屬性命名(**text vs text2 ; href vs href2**)，區分出那些資料是屬於新增的，而那些資料是屬於改和刪的，這邊是較複雜的地方，需要花點時間理解一下。

* 修改 `./backend/menu.php` 中編輯次選單按鈕連結及參數
* 在 `./modal/` 目錄下建立一個 `submenu.php` 的檔案做為編輯次選單的主要畫面
* 編輯 `./modal/submenu.php` 以符合參考圖的呈現格式
* 在 **更多次選單** 上加入 `onclick` 事件呼叫 `more()` 程式來動態產生輸入欄位
```html

    <input type="button" value="更多次選單" onclick="more()">

    <script>
      function more(){
          let row=`
              <tr>
                  <td><input type="text" name="name2[]"></td>
                  <td><input type="text" name="href2[]"></td>
                  <td></td>
              </tr>
          `
          $("#sub").append(row)
      }
    </script>

```
* 新增用的欄位名應該要和從資料庫撈出來的不一樣，才能做識別(ex. text vs text2)
* 新增 `./api/edit_submenu.php` 撰寫編輯次選單的功能
* 依照POST內容的欄位名稱來決定要執行的是新增或是修改或是兩者同時都有。
* 這邊我們採用表單送出的行為(submit)，也就是整個頁面會跳去 `./api/edit_submenu.php` 處理完再跳回 `admin.php?do=menu`，跳回來時不會再彈出視窗，但是可以看到次選單的數目改變。
* 如果希望保留彈出視窗，那麼就要改用AJAX的方式來撰寫程式。
* 記得要把主選單的**id**一併送出，才知道是誰的次選單，這邊我們使用 `hidden` 欄位來存放主選單id
```php
    <input type="hidden" name="parent" value="<?=$_GET['id'];?>">
```
* 修改 `./backend/menu.php` 中列表主選單的條件(`["parent"=>0]`)
* 最後補上次選單數的計算及顯示(使用$db->count()函式來計算次選單數)

---

## 步驟十一：製作分頁功能
本題組一共有三個地方會使用到分頁功能， `校園映像圖片`、`最新消息資料`、`更多消息`，其中兩個在後台，一個在前台，我們只需要做好一個分頁功能，利用變數的設定，就可以把程式碼複製給其它二個功能使用。

而在乙級的四個題組中，分頁功能的使用有三題，因此一定要熟悉分頁的製作方式。

* 先取得資料表中的總筆數(要注意是否有條件限制，比如全部列出或是只列出顯示設定為1的資料)
* 設定每個頁面要列出的資料筆數
* 計算總頁數(無條件進位法ceil()函式)
* 採用網址參數的方式來取得當前頁，預設為第一頁
* 計算資料的開始筆數( **(當前頁-1)*每頁筆數** )
* 下SQL查詢語法( **LIMIT start,amount** )
```php
  <?php
    $news=new DB("news");
    $total=$news->count(['sh'=>1]);
    $num=5;
    $pages=ceil($total/$num);
    $now=(!empty($_GET['p']))?$_GET['p']:1;
    $start=($now-1)*$num;
    $ns=$news->all(['sh'=>1]," limit $start,$num");
  ?>
```
* 列出資料
```php
  <?php
      foreach($ns as $n){
  ?>
    <li><?=mb_substr($n['text'],0,20,'utf8');?>...
        <div class='all' style="display:none"><?=$n['text'];?></div>
    </li> 
  <?php
    }
  ?>
```
* 製作下方分頁按鈕
```php
  <div style="text-align:center;">
    <?php if(($now-1)>0){ ?>
      <a class="bl" style="font-size:30px;" href="?do=new($now-1);?>">&lt;&nbsp;</a>
    <?php }  ?>
    <?php
      for($i=1;$i<=$pages;$i++){
          $fontsize=($i==$now)?'30px':'24px';
    ?>
     <a class="bl" style="font-size:<?=$fontsize;?>;" href="?do=news&p=<?=$i;?>"><?=$i;?></a>
    <?php } ?>
    <?php if(($now+1)<=$pages){ ?>
      <a class="bl" style="font-size:30px;" href="?do=news&p=<+1);?>">&nbsp;&gt;</a>
    <?php } ?>
  </div>
```
完成 `image`及`news` 的分頁製作後，後台的主要功能也完成90%了，剩下的是一些小調整，我們會放在前台製作時再一併處理。

---

## 步驟十二：處理登入功能及相關連結修改
題目並沒有很明確的指出登入功能一定要綁管理者帳號的資料表，所以這裹就算是寫死的，只要可以做到題目要求的登入後跳到後台頁面就可以了，但是這邊我們還是採用比較正常合理的方式來製作登入功能，因此我們要先在資料表中建立一個管理者帳號，帳號是"admin"，密碼是"1234"。

根據版型的暗示，前台的有一個登理登入的按鈕可以連到登入畫面，登入後到後台時有一個按鈕可以登出，而如果在登入的狀態下停在前台時，原本的管理登入按鈕會變成是可以跳到後台的按鈕，這表示我們需要有一個機制來記錄管理員的登入狀態，才能根據這個狀態對按鈕做出不同的功能，而後台的管理登出按鈕，題目只要求可以跳到管理登入的畫面，並沒有要求要做出真正的登出功能，比如清除session來登出，因此這邊也是先做到更改連結就可以了，時間足夠的話，可以把清除session的功能也做上。

登入及登出的按鈕功能在題目中並沒有特別的要求，如果正常解題時間來不及的話，可以不用處理按鈕的問題，先以符合題目要求的項目優先製作。

* 修改 `admin.php` 的管理登出按鈕中的onclick事件，我們這裹示範利用session登出的做法
* 撰寫 `./api/logout.php` 的登出功能，清除session後，要配合題目指示回到管理登入頁面(`index.php?do=login`)
* 在 `./front/login.php` 中撰寫登入功能，登入成功時增一個session來記錄登入狀態
* 題組一中並沒有要求要使用session或cookie來處理登入的功能，所以額外的功能建議是時間有餘時再來製作。
* 在 `indexlphp` 中修改管理登入連結的按鈕，依據session來決定要顯示的文字及連結內容

---

## 步驟十三：完成標題圖片/動態文字前台功能
前台的功能大多都是讀取資料，然後顯示出來，除了選單功能較複雜外，其他的功能就只是要細心點，別打錯字就可以了。

* 依題目要求，在 `header.php` 中取得`title`資料表中設定為顯示的資料，然後將相關的資訊放在頁面上的位置
* 記得修改原本版型的連結內容以符合我們的設計
* 由於我們已經使用`include`的功能把標題圖片切出去 `header.php` 檔了，所以完成 `header.php` 的修改時，`index.php`及`admin.php`的標題圖片都會同時生效
* 動態文字的標籤己經被我們切出去成為 `./front/marquee.php` ，因此我們只需要修改這個檔案即可
* 動態文字的效果需要使用迴圈的方式把文字輸出成一個字串來顯示，記得在資料之間加個空白或逗號來做出區隔

---

## 步驟十四：完成前台動畫圖片輪播及最新消息顯示
* 首頁的動畫及最新消息只需要修改 `./front/home.php` 
* 搬移動畫 `<script>` 的位置到動畫區塊後面，不然會發生畫面先空白三秒才開始輪播的問題
* 要在網頁載入後先寫入第一張動畫圖片，不然動畫區塊會先呈現空白
* 最新消息區要先判斷需要顯示的筆數是否超過五筆，決定是否會出現"more..."文字連結
* 處理最新消息的字串以配合內建JS的彈出視窗功能
* 在 `./front/news.php` 中處理最新消息顯示及分頁功能
* `news.php` 的消息清單改用 `<ol>` 來處理，在屬性`start`中可以填入序號起始值

---
  
## 步驟十五：完成校園映像檔輪播功能
版型檔案己經有提供了js程式來做圖片輪播的功能，不過程式本身有點小BUG，最後三張圖片有可能不會出現，這個問題可修可不修，現場評核時只看功能有沒有做出來，不會去驗證程式的小問題，所以我們要做的就是把圖片讀出來，然後讓JS程式去控制要顯示那幾張圖片。

內建程式的原理是先把圖片都讀出來後，都先設為隱藏，然後只顯示三張圖片，之後當點擊上下按鈕時，再去計算現在在第幾頁的位置，依此算出畫面上應該出現那三張圖片。

在讀出圖片時，需要在HTML的標籤中加入id序號來為每張圖片做記號，程式會去計算要顯示的圖片id，然後再使用 jQuery 的 `$(id).show()` 來讓圖片顯示出來。

內建程式問題出在計算當前頁的公式上，只要修改一下計算方式就可以正常輪播全部的圖片了。

---

## 步驟十六：完成選單功能
選單功能是前台功能中較為複雜的一個項目，因為有正副選單的關係，很多人會在這邊搞混，要特別注意巢狀迴圈應用上容易犯錯的地方；

另外，版型檔案雖然有提供了相關的JS及CSS，但是經過測試可能不是很完整，因為直接套用內建的css及js做出來的選單功能和題目示意圖的不太一樣，如果擔心會有爭議的話，可以再做一些修正，讓選單的顯示和題目的示意圖一致。

記得在後台配合題目要求將相關選單的項目先建好。

---

## 最後檢查
完成所有功能後，如果有時間，請記得完整的檢查一次所有的功能，尤其實是進站人數的功能需要關掉全部的瀏灠器再開，一定要確實檢查session有沒有正常運作。