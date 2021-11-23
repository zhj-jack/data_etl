# data_etl
'''
Created on 2018年11月26日

@author: zhouhaijie
'''

def csv_to_xlsx(url, turl):
    wb = Workbook()
    ws = wb.active
    with open(url, 'r') as f:
        for row in csv.reader(f):
            ws.append(row)
    wb.save(turl)


def check_coding(char_data):  # 判断字符编码类型
    if isinstance(char_data, bytes):
        pass
    else:
        char_data = char_data.encode()
    fencoding = chardet.detect(char_data)
    return fencoding['encoding']




def ins_pg(csvurl, tb, columns):
    p = time.time()
    if csvurl.endswith("xlsx") or csvurl.endswith("xls"):
        df = pd.read_excel(csvurl, header=None)
    else:
        if csvurl.endswith("csv"):
            cf = open(csvurl, "rb")
            d = cf.readline()
            encode = check_coding(d)
            cf.close()
            try:
                if encode == 'ascii':
                    encode = 'utf-8'
                '尝试利用pd方式读取,默认方式'
                df = pd.read_csv(csvurl, encoding=encode, header=None, escapechar='\\')
            except:
                try:
                    '尝试利用pd方式读取,python引擎解读'
                    df = pd.read_csv(csvurl, encoding=encode, header=None, sep=None, engine='python')
                except:
                    '尝试转换成excel'
                    turl = csvurl.replace(".csv", ".xlsx")
                    csv_to_xlsx(csvurl, turl)
                    df = pd.read_excel(turl, header=None)
        else:
            raise Exception("请上传.xlsx，.xls，.csv类型文件！")
        output = StringIO()
        df.iloc[1:].to_csv(output, sep='\t', index=False, header=False, escapechar='\\')
        output1 = output.getvalue()
        sta, msg = sqlConp.copy_from(StringIO(output1), tb, columns=columns)
        output.close()
        if sta == -1:
            raise Exception(msg)
        p1 = time.time()
