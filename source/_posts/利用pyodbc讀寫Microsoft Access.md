---
title: 利用pyodbc讀寫Microsoft Access
date: 2023-04-01 23:20:56
tags: 
- [兼職]
- [接案]
categories:
- [接案二三事]
---


## 緣起
最近接到一個小案子，案主是簡訊收發平台，他需要讀取本地資料庫的簡訊內容，增加客製字串後，再存入相同的資料庫。不難的case，但報酬不差，換算時薪約NT5,000，因此記錄一下案子內容。

## 讀取Microsoft Access
平台是Windows，使用的套件是pyodbc，使用前可能需安裝[Microsoft Access Database Engine 2010 可轉散發套件](https://www.microsoft.com/zh-tw/download/details.aspx?id=13255)

首先讀取mdb file，並製作轉換pattern的Dict

```python
# Set up DB info
default_MDB = "./SMS.mdb"
mdb = input("Please enter SMS.mdb path: ") or default_MDB
drv = "{Microsoft Access Driver (*.mdb, *.accdb)}"
pwd = "gd2013"

# Connect to DB
con = pyodbc.connect("DRIVER={};DBQ={};PWD={}".format(drv, mdb, pwd))
cur = con.cursor()

# Create dict of pattern
regex_pattern_list = parse_pattern()

# Create dict of direct pattern
pattern_replace_list = parse_direct_pattern()
```

之後，用pandas做資料處理，並將結果存回資料庫(mdb)

```python
while True:
    print(str(datetime.datetime.now()))
    # Query DB
    SQL = "SELECT * FROM L_SMS ORDER BY id"
    cur.execute(SQL)
    all_tuple = cur.fetchall()
    desc = cur.description

    # Read into dataframe
    print(f"Read database: {mdb} infomation")
    df_column = pd.DataFrame(desc)
    all_list = [list(i) for i in all_tuple]
    df = pd.DataFrame(all_list, columns=df_column[0])
        
    # Create dict of pattern
    for idx, x in enumerate(regex_pattern_list):
        x["id"] = idx            
        x["pattern_count"] = (
            df["content"]
            .str.contains(r"^{}".format(x["regex_to_be_replaced"]))
            .sum()
        )
        
    # Start to find and replace pattern
    for x in regex_pattern_list:
        # print(x['regex_to_be_replaced'])
        if x["pattern_count"] > 0:
            print(
                f"Found pattern: '{x['regex_to_be_replaced']}', Start replacing with '{x['regex_placed']}', count: {x['pattern_count']}..."
            )
            re_express = r"{}".format(x["regex_to_be_replaced"])
            re_express_1 = r"{}".format(x["regex_placed"])
            print(f"re_express: {re_express}")
            print(f"re_express_1: {re_express_1}")
            df["content"] = df["content"].str.replace(
                re_express, re_express_1, regex=True
            )

            # Write into database
            print(f"Write updated text into database: {mdb}")
            df_dict = df.to_dict()
            id_list = []
            content_list = []

            for k, v in df_dict["id"].items():
                id_list.append(v)

            for k, v in df_dict["content"].items():
                content_list.append(v)

            for id, content in zip(id_list, content_list):
                cur.execute(
                    "UPDATE L_SMS SET content = ? WHERE id = ?", content, id
                )

            con.commit()  # commit to database
            print("Normal pattern find and replace done!")
            # cur.close()
            # con.close()
            


        # print(direct_pattern["direct_pattern_count"])
        for idx, direct_pattern in enumerate(pattern_replace_list):
            direct_pattern["direct_pattern_count"] = (
            df["content"].str.contains(r"^{}".format(direct_pattern["pattern"])).sum()
            )
        
        # Start to find and replace direct pattern
        for direct_pattern in pattern_replace_list:
            # print(direct_pattern["direct_pattern_count"])
            # print(x['regex_to_be_replaced'])
            if direct_pattern["direct_pattern_count"] > 0 and direct_pattern['handle'] == False:
                print(
                    f"Found pattern: '{direct_pattern['pattern']}', Start replacing with '{direct_pattern['replaced']}', count: {direct_pattern['direct_pattern_count']}..."
                )
                re_express = r"{}".format(direct_pattern["pattern"])
                re_express_1 = r"{}".format(direct_pattern["replaced"])
                print(f"re_express: {re_express}")
                print(f"re_express_1: {re_express_1}")
                df["content"] = df["content"].str.replace(
                    re_express, re_express_1, regex=True
                )

                # Write into database
                print(f"Write updated text into database: {mdb}")
                df_dict = df.to_dict()
                id_list = []
                content_list = []

                for k, v in df_dict["id"].items():
                    id_list.append(v)

                for k, v in df_dict["content"].items():
                    content_list.append(v)

                for id, content in zip(id_list, content_list):
                    cur.execute(
                        "UPDATE L_SMS SET content = ? WHERE id = ?", content, id
                    )

                con.commit()  # commit to database
                print("Direct pattern find and replace done!")
                direct_pattern['handle'] = True
                # cur.close()
                # con.close()
        time.sleep(10)
```

## 執行結果

執行前，資料庫內有兩筆record符合我們的pattern
![](https://i.imgur.com/b1b0K9F.png)

執行後，可那兩筆record已經被修改
![](https://i.imgur.com/yf6UrMJ.png)

執行中
![](https://i.imgur.com/9yPxqEf.png)

[Github Repo完整程式碼](https://github.com/mulongfu/SMS)