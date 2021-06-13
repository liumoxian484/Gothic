# Gothic
java训练项目
import time
import requests
import pymysql


class Spider(object):
    def __init__(self):
        # 中间连接
        self.url = "https://search.51job.com/list/000000,000000,0000,00,9,99,%25E6%25B8%25B8%25E6%2588%258F%25E7%25AD%2596%25E5%2588%2592,2,{}.html?lang=c&postchannel=0000&workyear=99&cotype=99&degreefrom=99&jobterm=99&companysize=99&ord_field=0&dibiaoid=0&line=&welfare="
        # 存放链接的列表
        self.url_list = [self.url.format(i) for i in range(1, 64)]
        self.final_list = []
        self.headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) "
                                 "Chrome/89.0.4389.114 Safari/537.36",
                   "Accept": "application/json, text/javascript, */*; q=0.01"}  # 请求头，内有Accept
        self.params = {"lang": "c", "postchannel": "0000", "workyear": "99", "cotype": "99", "degreefrom": "99",
                  "jobterm": "99", "companysize": "99", "ord_field": "0", "dibiaoid": "0", "line": "",
                  "welfare": ""}  # 参数

    def get_key_dict(self):
        conn = pymysql.connect(host='127.0.0.1', user='root', password="root", database='51job', port=3306)
        cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)
        for i in self.url_list:
            respose = requests.get(url=i, headers=self.headers, params=self.params)
            params = respose.content
            if params:
                re = respose.json()
                qiu = respose.json().get('data')
                # print(qiu)
                re_list = re['engine_search_result']
                for i in re_list:  # 遍历所获之列表
                    tmp = i['attribute_text']  # 获取attribute_text列表之内容
                    try:
                        if tmp[2] == "初中及以下" or tmp[2] == "高中/中技/中专" or tmp[2] == "大专" or tmp[2] == "本科" or tmp[2] == "硕士" or tmp[2] == "博士":  # 对下标2之内容进行判断——下标2之内容通常为学历要求
                            edu = tmp[2]
                        else:
                            continue
                    except:
                        continue
                    tmpdict = {'name': i['job_name'], 'company': i['company_name'], 'salary': i['providesalary_text'],
                               'workarea': i['workarea_text'], 'education': edu}
                    # print(tmpdict)
                    self.final_list.append(tmpdict)
                try:
                    conn.begin()
                    for g in self.final_list:
                        cols = ", ".join('`{}`'.format(k) for k in g.keys())
                        # print(sql)
                        val_cols = ', '.join('"{}"'.format(k) for k in g.values())
                        fin_val = str(cols).replace("`", "")
                        # print(fin_val)
                        # e_val = eval(cols)
                        # print(val_cols, type(val_cols))  # '%(name)s, %(age)s'
                        sql = " insert into gameplaner(num,{}) values(null,{})".format(fin_val, val_cols)
                        cursor.execute(sql)
                        print(sql)
                        conn.commit()
                        # 提交
                except:
                    print("数据库连接错误。")
            else:
                print("有误，终止交易。")
                break
            # print(self.final_list)
            # for j in self.final_list:
            #     # time.sleep(2)
            #     print(j)

    def run(self):
        self.get_key_dict()


if __name__ == '__main__':
    spider = Spider()
    spider.run()
