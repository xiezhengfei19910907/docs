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

需要执行的操作如下:

目前宽屏的高度是508px, 窄屏的高度是390px. 更改 config中的size 参数.(其中宽屏是要改css的)

jjshouse.com/jjshouse.fr 需要插入宽屏的feature.



##### 1.2 sale分类相关

分类图片相关代码如下:

```html
{{ feature('/browse/1.0/browse_weeklydeal_light')|raw }}
```

```php
# 主要代码截取如下
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



##### 1.5 倒计时相关

具体的代码文件是 src/lisa/js/common/header.js 和 src/lisa/js/mod/countdown.js

主要代码截取如下:

```javascript
if ($('#weekly-deal-time')[0]) {
    var weekly_time = r.end_time - r.now_time;
    var weekly_deal_banner = new Countdown('#weekly-deal-time', weekly_time, 'days', false, true);
    weekly_deal_banner.run();
}
```

```javascript
if (this.isWeeklyDeal) {
    if (isDays) {
        var dayTxtClass = "weekly_days_txt";
    } else {
        var dayTxtClass = "weekly_day_txt";
    }
    _day = '<span class="weekly_day">' + _day + '</span>';
    _dayTxt = '<span class= ' + dayTxtClass + ' >' + _dayTxt + '</span>';
    _hour = '<span class="weekly_hour">' + _hour + '</span>';
    _minute = '<span class="weekly_minute">' + _minute + '</span>';
    _second = '<span class="weekly_second">' + _second + '</span>';

    var txt = _day + _dayTxt + _hour + _minute + _second;
}
```



#### 2. SALE商品列表页改动

##### 2.1 每个分类显示至少显示60个商品

具体weekly deal生成的逻辑在jjs_editor.git  cronjob/insert_weekly_deal_goods.php中, 可能需要改动逻辑来满足条件.**(当前分类销量从高到低排列)**



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

**jjshouse.com 和 jjshouse.fr**

倒计时banner的逻辑由下列feature控制.

```html
{{ feature('/header-slogan/1.0/slim-banner') | raw}}
```

需要修改配置, 使之在weekly deal 相关页面不显示.

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

**jjshouse.co.uk**

倒计时banner的逻辑由下列feature控制.

```html
{{ feature('/header-highlight/1.0/highlight')|raw }}
```

需要修改配置, 使之在weekly deal 相关页面不显示.

```json
{
    "widePagesCode": [
        "filter",
        "search",
        "cart"
    ],
    "black_pages": [
        "flash_sale",
        "promotion_sale",
        "bannerpage",
        "promotion_BlackFriday",
        "wholesale-weekly-deal"
    ],
    "processor": "prometheus\\header\\highlight\\HighLightProcessor",
    "tpl": "webapp\/common\/header\/highlight_extend.twig",
    "disableCountDown": "false"
}
```



##### 2.4 倒计时

需要新增feaure实现

```html
<div class="list_banner">
    <div class="list_narrow_banner">
        {{ feature("/banner/2.1/banner_weeklydeal_list")|raw }}
    </div>
    <div class="list_wide_banner">
        {{ feature("/banner/2.1/banner_weeklydeal_list_wide")|raw }}
    </div>
</div>
<span id="weekly-deal-list-time"></span>
```



##### 2.5 more items

在每页商品数量不满足60个的情况 和 当前分类只有60个商品的情况下显示more链接

```html
{% if weeklyDeal and ((ijk <= 59) or (total == 60) %}
    <div class="more_items">
        <a href="{{ catUrl }}" target="_blank" title="{{ catName }}">
            {{ 'page_gallery_shop'|nl }}{{ 'page_common_more'|nl }} {{ catName }} >>
        </a>
    </div>
{% endif %}
```



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









