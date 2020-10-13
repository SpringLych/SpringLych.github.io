---
title: 使用aiohttp进行异步爬虫
date: 2019-04-04 16:39:48
tags: [Python, 爬虫]
categories: Python
---

不久前出于兴趣需要爬取一个网站的图片，保存到本地。这里记录下使用aiohttp的过程。

<!--more-->

使用成熟的框架Scrapy爬虫时，其对于图片处理的不是很好，对于GIF也没有太大的支持。查找资料后发现了aiohttp这个框架。

## 单独使用aiohttp

使用BeautifulSoup解析HTML页面

```python
async def process_html(page_url):
    """解析文章页面，填充item"""
    async with aiohttp.ClientSession() as session:
        async with session.get(page_url) as response:
            cont = await response.read()
            soup = bs(cont, 'lxml')
            item['title'] = soup.find('h1').string
            item['page_url'] = page_url
            item['time'] = item['page_url'].split('/')[0]
            content = soup.find('div', class_ = 'main')
            data_src_temp = content.find_all(
                'img', attrs = {'type': 'image'})
            # 得到图片链接
            for link in data_src_temp:
                src_temp = link.get('image')
                item['images_url'].append(src_temp)
            time_path = os.path.join('imagepath', item['time'])
            page_path = set_down_path(time_path, item['title'])
            # print(item['images_url'])
            for idx, image_url in enumerate(item['images_url']):
                await down_one(image_url, page_path, idx)
                

async def down_one(img_url, page_path_temp, idx):
    """得到content后保存到本地"""
    # 图片后缀
    suffix = img_url.split('.')[-1]
    # 组装单个图片路径
    image_name = page_path_temp + "/" + str(idx) + '.' + suffix
    if os.path.exists(image_name):
        return

    # 得到图片内容
    async with aiohttp.ClientSession() as session:
        async with session.get(img_url) as response:
            content = await response.read()
            with open(image_name, 'wb') as file:
                file.write(content)
                
def run():
    print("start")
    t1 = time.time()
    loop = asyncio.get_event_loop()
    tasks = [process_html(url) for url in page_urls]
    loop.run_until_complete(asyncio.wait(tasks))
    loop.close()
    t2 = time.time()
    print('总共耗时：%s' % (t2 - t1))
```





## aiohttp+Scrapy

[aiohttp](<https://aiohttp.readthedocs.io/en/stable/>)是requests的异步替代版。

下面是结合Scrapy的代码，由Scrapy处理得到每页中每张图片的地址，在`pipelines.py`中使用aiohttp+asyncio下载图片。



```python
class SaveImagePipeline(object):
    def set_down_path(self, time_path, title):
        """
        设置图片的文件夹路径
        """
        try:
            if not os.path.exists(time_path):
                os.mkdir(time_path)
            page_path = os.path.join(time_path, title)
            if not os.path.exists(page_path):
                os.mkdir(page_path)
        except IOError as e:
            logging.error(e)
            return ''
        return page_path

    async def down_one(self, img_url, page_path, idx):
        """得到content后保存到本地"""
        # 图片后缀
        suffix = img_url.split('.')[-1]
        # 组装单个图片路径
        image_name = page_path + "/" + str(idx) + '.' + suffix
        if os.path.exists(image_name):
            return

        # 得到图片内容
        async with aiohttp.ClientSession() as session:
            try:
                async with session.get(img_url, verify_ssl = False) as response:
                    if response.status == 404:
                        return
                    content = await response.read()
                    with open(image_name, 'wb') as file:
                        file.write(content)
            except aiohttp.client_exceptions.ClientConnectionError:
                return

    tasks = ''

    def process_item(self, item, spider):
        post_time = item['time']
        title = item['title']
        time_path = os.path.join(settings['SAVE_IMAGE_PATH'], post_time)
        page_path = self.set_down_path(time_path, title)

        loop = asyncio.get_event_loop()
        self.tasks = [self.down_one(url, page_path, idx) for idx, url in enumerate(item['images_url'])]
        loop.run_until_complete(asyncio.wait(self.tasks))

```

如果图片更多，保存图片的操作就成了性能限制，可以尝试使用 [aiofiles](<https://github.com/Tinche/aiofiles>) 进行图片保存



## 参考

[Welcome to AIOHTTP](<https://aiohttp.readthedocs.io/en/stable/>)

[掘金-python使用异步每秒钟就能下载一张高清大图，快不快？](<https://juejin.im/post/5b496e2ce51d45195866d455>)

