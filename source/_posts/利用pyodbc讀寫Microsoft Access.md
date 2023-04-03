---
title: 利用pyodbc讀寫Microsoft Access
date: 2022-08-25 23:20:56
tags: 
- [兼職]
- [接案]
categories:
- [接案二三事]
---


## 緣起
最近接到一個小案子，案主是簡訊收發平台，他需要讀取本地資料庫的簡訊內容，增加客製字串後，再存入相同的資料庫。老實說是easy case，但報酬不差，換算時薪約NT3,000，因此記錄一下案子內容。

## 讀取Microsoft Access
平台是Windows，使用的套件是pyodbc，使用前可能需安裝[Microsoft Access Database Engine 2010 可轉散發套件](https://www.microsoft.com/zh-tw/download/details.aspx?id=13255)，以下是讀取.mdb檔

```python
# Set up DB info
DEFAULT_MDB = "./SMS.mdb"
MDB = input("Please enter SMS.mdb path: ") or DEFAULT_MDB
DRV = "{Microsoft Access Driver (*.mdb, *.accdb)}"
PWD = "gd2013"

# Connect to DB
con = pyodbc.connect("DRIVER={};DBQ={};PWD={}".format(DRV, MDB, PWD))
cur = con.cursor()
SQL = "SELECT * FROM L_SMS ORDER BY id"
cur.execute(SQL)
```
讀取之後，用pandas做資料處理
```python
all_tuple = cur.fetchall()
desc = cur.description

# Read into dataframe
print(f"Read database: {MDB} infomation")
df_column = pd.DataFrame(desc)
all_list = [list(i) for i in all_tuple]
df = pd.DataFrame(all_list, columns=df_column[0])

JKF_PATTERN = r"(^您的簡訊驗證碼為\s[0-9]{4,8}\s為了帳號安全，請勿分享給他人)"
JKF_REPLACE_PATTERN = r"jkf[\1]"
TANTAN_PARRENT = r"(^您的驗證碼為:\s[0-9]{4,8}\s\(此驗證碼10分鐘內有效\))"
TANTAN_PARRENT_FULL = r"(^您的驗證碼為：\s[0-9]{4,8}\s\（此驗證碼10分鐘內有效\）)"
TANTAN_REPLACE_PARRENT = r"探探[\1]"

jkf_pattern_count = df["content"].str.contains(r"^您的簡訊驗證碼為\s[0-9]{4,8}\s為了帳號安全，請勿分享給他人").sum()
tantan_pattern_count = df["content"].str.contains(r"^您的驗證碼為:\s[0-9]{4,8}\s\(此驗證碼10分鐘內有效\)").sum()
tantan_pattern_count_full = df["content"].str.contains(r"^您的驗證碼為：\s[0-9]{4,8}\s\（此驗證碼10分鐘內有效\）").sum()
```

## 寫入Microsoft Access
我們用Regular Expression找尋特定的字串並依照指定字串替換
最後，用commit()寫入到資料庫

```python
# Replace record
df["content"] = df["content"].str.replace(JKF_PATTERN, JKF_REPLACE_PATTERN, regex=True)
df["content"] = df["content"].str.replace(
    TANTAN_PARRENT, TANTAN_REPLACE_PARRENT, regex=True
)
df["content"] = df["content"].str.replace(
    TANTAN_PARRENT_FULL, TANTAN_REPLACE_PARRENT, regex=True
)

if jkf_pattern_count > 0 or tantan_pattern_count > 0 or tantan_pattern_count_full > 0:
    # Write into database
    print(f"Write updated text into database: {MDB}")
    df_dict=df.to_dict()
    id_list = []
    content_list = []

    for k, v in df_dict['id'].items():
        id_list.append(v)

    for k, v in df_dict['content'].items():
        content_list.append(v)

    for id, content in zip(id_list, content_list):

        cur.execute(
            "UPDATE L_SMS SET content = ? WHERE id = ?",
            content, id
        )

    con.commit() #commit to database
    print("Done!")

```

## 執行結果

執行前，資料庫內有兩筆record符合我們的pattern
![](https://i.imgur.com/b1b0K9F.png)

執行後，可那兩筆record已經被修改
![](https://i.imgur.com/yf6UrMJ.png)

執行中
![](https://i.imgur.com/9yPxqEf.png)

[Github Repo完整程式碼](https://github.com/mulongfu/update-microsoft-access)