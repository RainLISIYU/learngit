import json
import os
import zlib
import hashlib


class DiskCache:
    def __init__(self, cache_dir="cache", compress=True, encoding="utf8"):
        self.cache_dir = cache_dir
        self.compress = compress
        self.encoding = encoding

    # 将url转为hash码并作为文件名，文件中保存读取的数据
    def url_to_path(self, url):
        filename = hashlib.sha256(bytes(url, "utf8")).hexdigest()
        return os.path.join(self.cache_dir, filename)

    # 先将数据读取出来并解压
    def __getitem__(self, url):
        path = self.url_to_path(url)
        if os.path.exists(path):
            with open (path, "rb" if self.compress else "r") as file:
                if self.compress:
                    data = zlib.decompress(file.read()).decode(self.encoding)
                    print(json.loads(data))
                    return json.loads(data)
                return json.load(file)
        else:
            raise KeyError(f"{url} do not exist")

    # 将数据压缩后再存储
    def __setitem__(self, url, result):
        if not os.path.exists(self.cache_dir):
            os.mkdir(self.cache_dir)

        path = self.url_to_path(url)
        with open(path, "wb" if self.compress else "w") as file:
            if self.compress:
                data = bytes(json.dumps(result),self.encoding)
                file.write(zlib.compress(data))
            else:
                json.dump(result, file)
