from selenium import webdriver
import json
import time
from io import BytesIO
from PIL import Image
from selenium import webdriver
from selenium.common.exceptions import TimeoutException
from selenium.webdriver import ActionChains
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from os import listdir
from os.path import abspath, dirname
from selenium.common.exceptions import TimeoutException
from selenium.webdriver import ActionChains
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import requests
from hashlib import md5

PROXY_POOL_URL = 'http://localhost:5555/random'


class Chaojiying_Client(object):

    def __init__(self, username, password, soft_id):
        self.username = username
        self.password = md5(password.encode('utf8')).hexdigest()
        self.soft_id = soft_id
        self.base_params = {
            'user': self.username,
            'pass2': self.password,
            'softid': self.soft_id,
        }
        self.headers = {
            'Connection': 'Keep-Alive',
            'User-Agent': 'Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0)',
        }

    def PostPic(self, im, codetype):
        """
        im: 图片字节
        codetype: 题目类型 参考 http://www.chaojiying.com/price.html
        """
        params = {
            'codetype': codetype,
        }
        params.update(self.base_params)
        files = {'userfile': ('ccc.jpg', im)}
        r = requests.post('http://upload.chaojiying.net/Upload/Processing.php', data=params,
                          files=files, headers=self.headers)
        return r.json()

    def ReportError(self, im_id):
        """
        im_id:报错题目的图片ID
        """
        params = {
            'id': im_id,
        }
        params.update(self.base_params)
        r = requests.post('http://upload.chaojiying.net/Upload/ReportError.php', data=params,
                          headers=self.headers)
        return r.json()


class CET(object):
    def __init__(self, name, cet):
        self.chaojiying = Chaojiying_Client('wkaanig721', '123TMD456', '96001')
        self.browser = webdriver.Chrome()
        self.wait = WebDriverWait(self.browser, 20)
        self.url = 'https://www.chsi.com.cn/cet/'
        self.name = name
        self.num = self.num()  # list
        self.schoolcode = '412020'
        self.month = '1'  # 1是上半年2是下半年
        if cet == '4':
            self.cet = '1'
        elif cet == '6':
            self.cet = '2'  # 1是四级2是六级
        self.firstnum = self.schoolcode + '18' + self.month + self.cet

    def get_proxy(self):
        try:
            response = requests.get(PROXY_POOL_URL)
            if response.status_code == 200:
                return response.text
            return None
        except ConnectionError:
            return None

    def open(self):
        """
        chromeOptions = webdriver.ChromeOptions()
        chromeOptions.add_argument('--proxy-server=http://' + self.get_proxy())
        self.browser = webdriver.Chrome(chrome_options=chromeOptions)
        """
        self.browser.get(self.url)

    def testnum(self, number):
        print(number)
        num = self.wait.until(EC.presence_of_element_located((By.CSS_SELECTOR, '#zkzh')))
        name = self.wait.until(EC.presence_of_element_located((By.CSS_SELECTOR, '#xm')))
        num.send_keys(number)
        name.send_keys(self.name)

    def num(self):
        classes = ['%03d' % c for c in range(73, 75)]
        sitnum = ['%02d' % s for s in range(25, 27)]
        numlist = [''.join([c, s]) for c in classes for s in sitnum]
        print(numlist)
        return numlist

    def codeimage(self):
        yzm = self.wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR, '# yzm')))
        yzm.click()

        im = self.wait.until(EC.presence_of_element_located((By.CSS_SELECTOR, '#stu_reg_vcode')))
        yzm.send_keys(self.chaojiying.PostPic(im, 1902))
        submit = self.wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR, '# submitCET')))
        submit.click()

    def login_successfully(self):
        """
        判断是否登录成功
        :return:
        """
        try:
            return bool(
                WebDriverWait(self.browser, 5).until(EC.text_to_be_present_in_element(
                    (By.CSS_SELECTOR, '# leftH > div > table > tbody > tr:nth-child(1) > td'),
                    self.name)))
        except TimeoutException:
            return False

    def num_error(self):
        """
        判断是否准考证错误
        :return:
        """
        try:
            return WebDriverWait(self.browser, 5).until(
                EC.text_to_be_present_in_element(
                    (By.CSS_SELECTOR, '#leftH > div > div.error.alignC.marginT20'),
                    '无法找到对应的分数，请确认您输入的准考证号及姓名无误。'))
        except TimeoutException:
            return False

    def yzm_error(self):
        """
        判断是否验证码错误
        :return:
        """
        try:
            return WebDriverWait(self.browser, 5).until(
                EC.text_to_be_present_in_element((By.CSS_SELECTOR, '# form1 > div'), '验证码不正确'))
        except TimeoutException:
            return False

    def find(self):
        for testnum in self.num:
            self.testnum(testnum)
            self.codeimage()

    def main(self):
        self.open()
        for testnum in self.num:
            num = self.firstnum + testnum
            self.testnum(num)
            self.codeimage()
            if self.yzm_error():
                self.testnum(num)
                self.codeimage()
            else:
                return {
                    'status': 3,
                    'content': '验证码二次错误'
                }
            if self.num_error():
                return {
                    'status': 2,
                    'content': '准考证号错误'
                }

            if self.login_successfully():
                return {
                    'status': 1,
                    'content': '查询成功' + num + self.name
                }


if __name__ == '__main__':
    cet = CET('王X', '4')
    cet.main()
