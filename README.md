# データベースー正規化レク

## 関数従属
正規化の話をする前に、関数従属について説明します。
> 関数従属（functional dependency）は、データベース設計において重要な概念で、正規化を理解するために欠かせません。関数従属とは、ある属性の集合が他の属性の集合を一意に決定する関係性を指します。

### 関数従属性
- 属性セット 𝑋 が属性セット 𝑌 を関数従属させる場合、「𝑋→𝑌」と表現します。
- これは、テーブル内の任意の2つのレコードにおいて、𝑋 の値が同じならば、𝑌 の値も必ず同じであるという関係を意味します。

次のようなテーブルを考えてみましょう。
| 学生ID | 学生名   | 学科       |
|--------|----------|------------|
| 1      | 山田太郎 | コンピュータ |
| 2      | 佐藤花子 | 物理       |
| 3      | 山田太郎 | 数学       |

このテーブルにおいて、以下の関数従属が成り立つ場合があります。
- 学生ID→学生名

しかし、次の関数従属は成り立ちません。
- 学生名 → 学生ID

### 推移的関数従属
推移的関数従属は、正規化の第三正規形（3NF）で解消すべき問題です。具体的には、属性Aが属性Bを関数従属させ、属性Bが属性Cを関数従属させる場合、属性Aが属性Cを間接的に関数従属させることを意味します。

例として以下のようなテーブルを考えます。
```mermaid
erDiagram
    Student {
        int studentID
        string department
        string headOfDepartment
    }
    Student {
        1 "Computer Science" "Mr. Tanaka"
        2 "Physics" "Mr. Suzuki"
        3 "Mathematics" "Mr. Sato"
    }
```

このテーブルにおいて、以下の関数従属が成り立つ場合があります。
- 学生ID(A)→学科(B)
- 学科(B)→学科主任(C)

したがって、学生ID → 学科主任という推移的関数従属が成り立ちます。これを解消するためには、テーブルを分割します。
```mermaid
erDiagram
    Student {
        int studentID
        string department
    }
    Department {
        string department
        string headOfDepartment
    }
    Student {
        1 "Computer Science"
        2 "Physics"
        3 "Mathematics"
    }
    Department {
        "Computer Science" "Mr. Tanaka"
        "Physics" "Mr. Suzuki"
        "Mathematics" "Mr. Sato"
    }
```

このようにして、推移的関数従属を解消することでデータベースの正規化が達成されます。

## 正規化とは
>データベースの正規化は、データの重複を最小限に抑え、データの一貫性と整合性を維持するためのデータベース設計手法です。これにより、データの更新や検索が効率的になり、データの整合性が保たれます。

| 学生ID | 学生名  | コース      | 成績 |
|--------|---------|--------------|--------------|
| 1      | 山田太郎| 数学, 物理   | A, B         |
| 2      | 佐藤花子| 英語         | B            |
| 3      | 鈴木一郎| 数学, 英語 | A, B         |

このテーブルには冗長性と不整合のリスクがあります。これを第一正規形（1NF）から第三正規形（3NF）まで正規化していきます。

### 第一正規形（1NF）
**要件**
- 各列が原子的であること（各セルに単一の値が入っていること）。

原子的であるとは、各セルに一つの値だけが入っている状態を指します。つまり、セルの中に複数の値やリストのようなものが入っていないことを意味します。

```mermaid
erDiagram
    StudentGrade {
        int studentID
        string studentName
        string course
        string grade
    }
    StudentGrade {
        1 "Yamada Taro" "Mathematics" "A"
        1 "Yamada Taro" "Physics" "B"
        2 "Sato Hanako" "English" "B"
        3 "Suzuki Ichiro" "Mathematics" "A"
        3 "Suzuki Ichiro" "English" "B"
    }
```

### 第二正規形（2NF）
**要件**
- 1NFに従っていること。
- 部分関数従属がないこと（非キー属性が部分的にキー属性に依存しないこと）。

ここで、次の部分関数従属が存在することに注意します:
- 学生名は学生IDにのみ依存します（部分関数従属）。

これを解消するために、テーブルを分割します。

```mermaid
erDiagram
    Student {
        int studentID
        string studentName
    }
    Grade {
        int studentID
        string course
        string grade
    }
    Student ||--o{ Grade: "has"
```

### 第三正規形（3NF）
**要件**
- 2NFに従っていること。
- 非キー属性が他の非キー属性に依存しないこと（推移的関数従属がないこと）。

ここで、次の推移的関数従属が存在することに注意します:
- コースに関する情報が他のテーブルに依存している場合、それを分離します。

```mermaid
erDiagram
    Student {
        int studentID
        string studentName
    }
    Grade {
        int studentID
        int courseID
        string grade
    }
    Course {
        int courseID
        string courseName
    }
    Student ||--o{ Grade: "has"
    Course ||--o{ Grade: "includes"
```
