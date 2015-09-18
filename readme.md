# Lab5 - 書籍折扣計算

目的：學習如何使用 PHPUnit 進行 TDD 開發

## 規格

* 歐你來系列書籍目前每本價格皆為 500 元
* 如果同時買兩本不同書籍，將享有 95 折優惠，最後金額為 (500 + 500) * 0.95 = 950
* 如果同時買三本不同書籍，將享有 9 折優惠，最後金額為 (500 + 500 + 500) * 0.9 = 1350
* 如果同時買四本不同書籍，將享有 8 折優惠，最後金額為 (500 + 500 + 500 + 500) * 0.8 = 1600
* 如果同時買五本不同書籍，將享有 75 折優惠，最後金額為 (500 + 500 + 500 + 500 + 500) * 0.75 = 1875
* 如果買了三本，但只有兩本不同，其中一本是重複的，那麼只有不同的兩本書能以 95 折購買，另一本要以原價 500 元計算。最後金額為： (500 + 500) * 0.95 + 500 = 1450
* 如果買了四本，但只有三本不同，其中一本是重複的，那麼只有不同的三本書能以 9 折購買，另一本要以原價 500 元計算。最後金額為： (500 + 500 + 500) * 0.9 + 500 = 1850

## 系統基本設計

書店裡會有書籍 (`Book`) ，並有一個折扣計算機 (`DiscountCalculator`) ，可以輸入書籍資訊 (`addBook`) ，並計算 (`calculte`) 出最後金額。

## 建立測試類別

建立 `tests/DiscountCalculatorTest.php` ，內容如下：

```php
<?php

use Lab5\DiscountCalculator;

class DiscountCalculatorTest extends PHPUnit_Framework_TestCase
{
    /**
     * @var DiscountCalculator
     */
    protected $discountCalculator;

    protected function setUp()
    {
        $this->discountCalculator = new DiscountCalculator();
    }
}
```

## 加入測試案例

加入第一個測試案例：買一本沒有折扣。

```php

    /**
     * 歐你來系列書籍目前每本價格皆為 500 元
     * 買一本沒有折扣
     * 最後金額為 500 * 1.0 = 500
     *
     * @test
     */
    public function buy_1_book()
    {
        // Arrange
        $expected = 500;

        // Act
        $this->discountCalculator->addBook(new Book('Book1', 500));
        $this->discountCalculator->calculate();
        $actual = $this->discountCalculator->getTotal();

        // Assert
        $this->assertEquals($expected, $actual);
    }
```

執行測試，紅燈。

## 加入目標類別，並通過第一個測試案例

建立 `src/DiscountCalculator.php` ，內容如下：

```php
<?php

namespace Lab5;

class DiscountCalculator
{
    public function addBook(Book $book)
    {
    }

    public function calculate()
    {
    }

    public function getTotal()
    {
        return 500;
    }
}
```

建立 `src/Book.php` ，內容如下：

```php
<?php

namespace Lab5;

class Book
{

}
```

執行測試，綠燈。

## 加入測試案例

加入第二個測試案例：如果同時買兩本不同書籍，將享有 95 折優惠：

```php
    /**
     * 如果同時買兩本不同書籍，將享有 95 折優惠
     * 最後金額為 (500 + 500) * 0.95 = 950
     *
     * @test
     */
    public function buy_2_books()
    {
        // Arrange
        $expected = 950;

        // Act
        $this->discountCalculator->addBook(new Book('Book1', 500));
        $this->discountCalculator->addBook(new Book('Book2', 500));
        $this->discountCalculator->calculate();
        $actual = $this->discountCalculator->getTotal();

        // Assert
        $this->assertEquals($expected, $actual);
    }
```

執行測試，紅燈。

## 修改目標類別以通過測試

`Book` 類別改為：

```php
class Book
{
    private $name;
    private $price;

    public function __construct($name, $price)
    {
        $this->name = $name;
        $this->price = $price;
    }

    public function __get($name)
    {
        if (property_exists($this, $name)) {
            return $this->$name;
        }
    }
}
```

`DiscountCalculator` 類別修改為：

```php
class DiscountCalculator
{
    protected $books = [];

    protected $total = 0;

    public function addBook(Book $book)
    {
        $this->books[] = $book;
    }

    public function calculate()
    {
        $this->total = 0;

        foreach ($this->books as $book) {
            $this->total += $book->price;
        }

        if (2 === count($this->books)) {
            $this->total *= 0.95;
        }
    }

    public function getTotal()
    {
        return $this->total;
    }
}
```

執行測試，綠燈。

## 加入第三個測試案例：如果同時買三本不同書籍，將享有 9 折優惠

在 `DiscountCalculatorTest` 類別中加入：

```php
    /**
     * 如果同時買三本不同書籍，將享有 9 折優惠
     * 最後金額為 (500 + 500 + 500) * 0.9 = 1350
     *
     * @test
     */
    public function buy_3_books()
    {
        // Arrange
        $expected = 1350;

        // Act
        $this->discountCalculator->addBook(new Book('Book1', 500));
        $this->discountCalculator->addBook(new Book('Book2', 500));
        $this->discountCalculator->addBook(new Book('Book3', 500));
        $this->discountCalculator->calculate();
        $actual = $this->discountCalculator->getTotal();

        // Assert
        $this->assertEquals($expected, $actual);
    }
```

執行測試，紅燈。

## 修改目標類別以通過測試

```php
    if (2 === count($this->books)) {
        $this->total *= 0.95;
    } elseif (3 === count($this->books)) {
        $this->total *= 0.9;
    }
```

執行測試，綠燈。

## 重構，提煉方法

將 `calculate` 方法中的以下程式碼：

```php
    $this->total = 0;

    foreach ($this->books as $book) {
        $this->total += $book->price;
    }
```

提煉成 `calculateSum` 方法：

```php
    $this->calculateSum();
```

```php
    private function calculateSum()
    {
        $this->total = 0;
        foreach ($this->books as $book) {
            $this->total += $book->price;
        }
    }
```

將 `calculate` 方法中的以下程式碼：

```php
    if (2 === count($this->books)) {
        $this->total *= 0.95;
    } elseif (3 === count($this->books)) {
        $this->total *= 0.9;
    }
```

提煉成 `calculateDiscount` 方法：

```php
    $this->calculateDiscount();
```

```php
    private function calculateDiscount()
    {
        if (2 === count($this->books)) {
            $this->total *= 0.95;
        } elseif (3 === count($this->books)) {
            $this->total *= 0.9;
        }
    }
```

## 加入第四個測試案例：如果同時買四本不同書籍，將享有 8 折優惠

在 `DiscountCalculatorTest` 類別中加入：

```php

    /**
     * 如果同時買四本不同書籍，將享有 8 折優惠
     * 最後金額為 (500 + 500 + 500 + 500) * 0.8 = 1600
     *
     * @test
     */
    public function buy_4_books()
    {
        // Arrange
        $expected = 1600;

        // Act
        $this->discountCalculator->addBook(new Book('Book1', 500));
        $this->discountCalculator->addBook(new Book('Book2', 500));
        $this->discountCalculator->addBook(new Book('Book3', 500));
        $this->discountCalculator->addBook(new Book('Book4', 500));
        $this->discountCalculator->calculate();
        $actual = $this->discountCalculator->getTotal();

        // Assert
        $this->assertEquals($expected, $actual);
    }
```

執行測試，紅燈。

## 修改目標類別以通過測試

修改 `calculateDiscount` 方法，在 `if ... else` 最後加入：

```php
    elseif (4 === count($this->books)) {
        $this->total *= 0.8;
    }
```

執行測試，綠燈。

## 加入第五個測試案例：如果同時買五本不同書籍，將享有 75 折優惠

在 `DiscountCalculatorTest` 類別中加入：

```php
    /**
     * 如果同時買五本不同書籍，將享有 75 折優惠
     * 最後金額為 (500 + 500 + 500 + 500 + 500) * 0.75 = 1875
     *
     * @test
     */
    public function buy_5_books()
    {
        // Arrange
        $expected = 1875;

        // Act
        $this->discountCalculator->addBook(new Book('Book1', 500));
        $this->discountCalculator->addBook(new Book('Book2', 500));
        $this->discountCalculator->addBook(new Book('Book3', 500));
        $this->discountCalculator->addBook(new Book('Book4', 500));
        $this->discountCalculator->addBook(new Book('Book5', 500));
        $this->discountCalculator->calculate();
        $actual = $this->discountCalculator->getTotal();

        // Assert
        $this->assertEquals($expected, $actual);
    }
```

執行測試，紅燈。

## 修改目標類別以通過測試

修改 `calculateDiscount` 方法，在 `if ... else` 最後加入：

```php
    elseif (5 === count($this->books)) {
        $this->total *= 0.75;
    }
```

執行測試，綠燈。

## 重構，提煉變數

修改 `calculateDiscount` 方法，先把 `count($this->books)` 改成區域變數。

選取 `count($this->books)` ，按 `Ctrl` `T` ，選擇 `Extract Variable...` ，變數名稱為 `uniqueCount` ：

```php
    $uniqueCount = count($this->books);

    if (2 === $count) {
        $this->total *= 0.95;
    } elseif (3 === $count) {
        $this->total *= 0.9;
    } elseif (4 === $count) {
        $this->total *= 0.8;
    } elseif (5 === $count) {
        $this->total *= 0.75;
    }
```

執行測試，綠燈。

## 加入第六個測試案例：如果買了三本，但只有兩本不同，其中一本是重複的

在 `DiscountCalculatorTest` 類別中加入：

```php
    /**
     * 如果買了三本，但只有兩本不同，其中一本是重複的，
     * 那麼只有不同的兩本書能以 95 折購買，另一本要以原價 500 元計算。
     * 最後金額為： (500 + 500) * 0.95 + 500 = 1450
     *
     * @test
     */
    public function buy_3_books_but_1_duplicate()
    {
        // Arrange
        $expected = 1450;

        // Act
        $this->discountCalculator->addBook(new Book('Book1', 500));
        $this->discountCalculator->addBook(new Book('Book2', 500));
        $this->discountCalculator->addBook(new Book('Book1', 500));
        $this->discountCalculator->calculate();
        $actual = $this->discountCalculator->getTotal();

        // Assert
        $this->assertEquals($expected, $actual);
    }
```

執行測試，紅燈。

## 修改目標類別以通過測試

新增一個 `$uniqueBooks` 屬性：

```php
    private $uniqueBooks = [];
```

修改 `calculateSum` 方法，比對沒有在 `uniqueBooks` 中的書籍才加總：

```php
    foreach ($this->books as $book) {
        if (!in_array($book->name, $this->uniqueBooks)) {
            $this->total += $book->price;
            $this->uniqueBooks[] = $book->name;
        }
    }
```

修改 `calculateDiscount` 方法，把 `$this->book` 改為 `$this->uniqueBooks` ：

```php
    $uniqueCount = count($this->uniqueBooks);
```

將以下程式加在 `calculateDiscount` 方法最後面：

```php
    $this->total += 500 * (count($this->books) - $uniqueCount);
```

執行測試，綠燈。

## 重構，提煉方法

將 `calculateDiscount` 方法中的：

```php
    if (2 === $count) {
        $this->total *= 0.95;
    } elseif (3 === $count) {
        $this->total *= 0.9;
    } elseif (4 === $count) {
        $this->total *= 0.8;
    } elseif (5 === $count) {
        $this->total *= 0.75;
    }
```

提煉成 `getDiscountByCount` 方法：

```php
    $this->getDiscountByCount($uniqueCount);
```

```php
    /**
     * @param $uniqueCount
     */
    private function getDiscountByCount($uniqueCount)
    {
        if (2 === $uniqueCount) {
            $this->total *= 0.95;
        } elseif (3 === $uniqueCount) {
            $this->total *= 0.9;
        } elseif (4 === $uniqueCount) {
            $this->total *= 0.8;
        } elseif (5 === $uniqueCount) {
            $this->total *= 0.75;
        }
    }
```

執行測試，綠燈。

## 重構，調整演算法

將 `getDiscountByCount` 方法中的：

```php
    if (2 === $uniqueCount) {
        $this->total *= 0.95;
    } elseif (3 === $uniqueCount) {
        $this->total *= 0.9;
    } elseif (4 === $uniqueCount) {
        $this->total *= 0.8;
    } elseif (5 === $uniqueCount) {
        $this->total *= 0.75;
    }
```

改為：

```php
    $discount = [
        1 => 1.0,
        2 => 0.95,
        3 => 0.9,
        4 => 0.8,
        5 => 0.75,
    ];
    $this->total *= $discount[$uniqueCount];
```

## 加入第七個測試案例：如果買了四本，但只有三本不同，其中一本是重複的


在 `DiscountCalculatorTest` 類別中加入：

```php
    /**
     * 如果買了四本，但只有三本不同，其中一本是重複的，
     * 那麼只有不同的三本書能以 9 折購買，另一本要以原價 500 元計算。
     * 最後金額為： (500 + 500 + 500) * 0.9 + 500 = 1850
     *
     * @test
     */
    public function buy_4_books_but_1_duplicate()
    {
        // Arrange
        $expected = 1850;

        // Act
        $this->discountCalculator->addBook(new Book('Book1', 500));
        $this->discountCalculator->addBook(new Book('Book2', 500));
        $this->discountCalculator->addBook(new Book('Book3', 500));
        $this->discountCalculator->addBook(new Book('Book1', 500));
        $this->discountCalculator->calculate();
        $actual = $this->discountCalculator->getTotal();

        // Assert
        $this->assertEquals($expected, $actual);
    }
```

執行測試，綠燈。
