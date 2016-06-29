# sale 页面改版记录



#### 1. sale 页面改动

##### 1.1 banner相关

sale页面banner(**webapp/promotion/weekly_deal/banner.twig**)使用的是

```h t m l
{{ feature("/banner/2.1/banner_weeklydeal_day#{month_day}")|raw }}
```

其中

```php
$month_day = date('j') % 5 + 1;	// j, 月份中的第几天, 没有前导零, 1 到 31 
```



查看banner配置如下:

```json
{
    "processor": "lestore\\banner\\BannerProcessor",
    "tpl": "banner.htm",
    "pattern": "",
    "size": {
        "height": 176,
        "width": 990
    },
    "time": {
        "start": "",
        "end": ""
    },
    "urlRoot": "http:\/\/d3mna48k5fyuxs.cloudfront.net\/v5res",
    "imgs": [
        {
            "id": "0",
            "enable": {
                "en": true,
                "de": true,
                "fr": true,
                "es": true,
                "se": true,
                "no": true,
                "it": true,
                "pt": true,
                "da": true,
                "fi": true,
                "ru": true,
                "nl": true,
                "ja": true
            },
            "title": [],
            "url": "jjshouse\/2016-06-02\/images\/banners\/weeklydealbanner\/weeklydeal1\/1{lang}.jpg",
            "active": true,
            "areas": []
        },
        {
            "id": "1",
            "enable": {
                "en": true,
                "de": true,
                "fr": true,
                "es": true,
                "se": true,
                "no": true,
                "it": true,
                "pt": true,
                "da": true,
                "fi": true,
                "ru": true,
                "nl": true
            },
            "title": [],
            "url": "jjshouse\/2015-02-02\/images\/banners\/wd1\/weekly-deal_{lang}.jpg",
            "active": false,
            "areas": []
        }
    ],
    "effect": []
}
```

根据产品的需求, banner尺寸日后可能会调整.调整size参数即可.页面会自适应. 其中$deal_time时间样式可能需要调整.



##### 1.2 sale分类相关

分类图片相关代码如下:

```html
{{ feature('/browse/1.0/browse_weeklydeal_light')|raw }}
```

```php
public function process($params = array()){
        global $container;
        foreach ($params as $key => $param) {
            $$key = $param;
        }
        if (isset($lang)) {
            $this->lang = $lang;
        }
        if (!$this->isActive()) {
            return false;
        }

        $config = json_decode($this->instance->config, true);
        if (!empty($config['catIds'])) {
            $this->catIds = $config['catIds'];
        }

        $categoryService = $container['category'];
        $list = array();

        foreach ($this->catIds as $_catId) {
            $list[$_catId] = $this->formatCatNode($catTree->getTreeNode($_catId));
        }
        foreach ($list as $key => $value) {
            $list[$key]['url'] .= 'weekly-deal/';
            // get category weekly-deal total
            $resource = new CategoryResource(null, array(), $key, $list[$key]['url']);
            $resource->extraFilters[CategoryResource::EXTRA_WEEKLY_DEAL] = true;
            $plistQuery = $resource->buildQuery($lang, 1);
            $plist = $container['plist']->get(new SearchListDesc($plistQuery));
            $list[$key]['total'] = $plist->total();
            $list[$key]['pic_path'] = $IMG_PATH . "wholesale-weekly-deal/{$key}.jpg";
        }
        $this->list = $list;

        $this->title = Helper::nl('page_common_weekly_deal');

        return true;
    }
```

其中feature config如下:

```json
{
    "processor": "lestore\\promotion\\browse\\CleanBrowseProcessor",
    "catIds": [
        2,
        4,
        7,
        8,
        3,
        5,
        89,
        132,
        133,
        84
    ]
}
```

分类图片途径示例: public/lisa/images/wholesale-weekly-deal/2.jpg

**如果产品想要更好的图片, 请提供图片, 图片是在代码仓库中的.**



#### 2. SALE商品列表页改动

##### 2.1 每个分类显示47个商品

具体weekly deal生成的逻辑在jjs_editor.git  cronjob/insert_weekly_deal_goods.php中







#### 3.SALE商品详情页改动





#### 4.其他改动

##### 4.1 鼠标悬浮在导航栏SALE分类时不再显示SPECIAL OFFER一列, 仅保留CATEGORIES.

feature: nav/1.0/header_light

需要删除如下内容即可.

```json
,
"promotion": [
  {
    "cat_id": 3,
    "filter": "/pmin0/pmax100",
    "name": {
      "fn": "price",
      "params": {
        "code": "page_common_dresses_under",
        "price": 100
      }
    }
  },
  {
    "cat_id": 5,
    "name": {
      "fn": "lang",
      "params": {
        "code": "page_common_sale_all_accessories"
      }
    }
  },
  {
    "cat_id": 33,
    "name": {
      "fn": "lang",
      "params": {
        "code": "page_common_fabric_swatch"
      }
    }
  }
]
```









