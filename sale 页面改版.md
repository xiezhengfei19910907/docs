# sale 页面改版记录



#### 1. sale 页面改动

##### 1.1 banner相关

sale页面banner(**webapp/promotion/weekly_deal/banner.twig**)使用的是

```h t m l
{{ feature("/banner/2.1/banner_weeklydeal_day#{month_day}")|raw }}
{{ feature("/banner/2.1/banner_weeklydeal_day#{month_day}_wide")|raw }}
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



##### 1.3 倒计时banner

具体内容见2.3 倒计时banner



##### 1.4 宽窄屏

```php
$web_container["/widescreen/1.0/hera_screen"] = array('page_code', "extra_page_code");
$web_container['extra_page_code'] = 'weekly_deal';
```

需要在对应的feature config 中添加 "extraPages":

```json
{
    "pages": [
        "index",
        "filter",
        "search",
        "cart",
        "orders",
        "order",
        "favorites",
        "tickets",
        "inquiries",
        "addressbook",
        "user",
        "ticket",
        "mycoupons"
    ],
    "bannerCatIds": [
        "5",
        "11",
        "132",
        "133",
        "89"
    ],
    "extraPages": [
        "weekly_deal"
    ],
    "processor": "prometheus\\widescreen\\WideScreenProcessor"
}
```





#### 2. SALE商品列表页改动

##### 2.1 每个分类显示至少显示59个商品

具体weekly deal生成的逻辑在jjs_editor.git  cronjob/insert_weekly_deal_goods.php中, 可能需要改动逻辑来满足条件.



##### 2.2 面包屑相关

对应feature是 

```html
{{ feature('/breadcrumb/1.0/cat_common')|raw }}
```

具体process代码在prometheus中已经覆写. 需要更新feature config 如下:

```json
{"processor":"prometheus\\category\\breadcrumb\\PrometheusBreadCrumbProcessor"}
```

需要执行如下sql用于更新语言包.

```sql
# backup
select * from multilanguage where code = 'page_common_sale1';

# exec
UPDATE multilanguage
SET en = 'Sale',
 de = 'Sale',
 fr = 'Soldes',
 es = 'Oferta',
 se = 'Rea',
 NO = 'Salg',
 it = 'Saldi',
 pt = 'Promoção',
 da = 'Salg',
 fi = 'Tarjous',
 ru = 'Скидки',
 nl = 'Sale',
 ar = 'تخفيض',
 be = 'Sale',
 hr = 'Sale',
 cs = 'Sleva',
 et = 'Sale',
 el = 'Sale',
 ht = 'Sale',
 he = 'Sale',
 hu = 'Sale',
 `is` = 'Sale',
 ga = 'Sale',
 ja = 'Sale',
 ko = 'Sale',
 lt = 'Sale',
 ms = 'Sale',
 mt = 'Sale',
 pl = 'Wyprzedaż',
 sk = 'Sale',
 sl = 'Sale',
 tr = 'İndirim'
WHERE
	CODE = 'page_common_sale1';
```



##### 2.3 倒计时banner

原先倒计时的逻辑由下列feature控制.

```html
{{ feature('/header-slogan/1.0/slim-banner') | raw}}
```

并修改了配置, 使之在weekly deal 相关页面不显示.

```json
{
    "black_list": [
        "index",
        "promotion_BlackFriday",
        "wholesale-weekly-deal"
    ],
    "banner_lang": [],
    "processor": "prometheus\\header\\slogan\\SlimBannerSloganProcessor",
    "show_timer": "true",
    "close_text": "true"
}
```

此处需要新加banner来满足weekly deal 的需求.



#### 3.SALE商品详情页改动

##### 3.1 在商品默认图片的左上角添加一个weekly deal 标签.

具体商品详情页的默认图由下列feature实现.

```html
{{feature("/product-picture/1.0/thumb-picture/original")|raw}}
```

判断商品是否参加当前的 weekly deal 活动, 如果参加, 显示对应的折扣标签.

**此处因为商品详情页改版,待改版完成后,再开发.**



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









