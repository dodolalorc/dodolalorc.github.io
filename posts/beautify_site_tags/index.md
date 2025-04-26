# 【网站美化】标签美化

## 简单设计

`assets/css/common/terms.css`添加：

```css
.terms-tags{
    padding-left: 0;
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    justify-content: center;
}
```

## 添加权重

在`./layouts/_default/terms.html`中，找到这段代码并在a标签插入`data-weight`属性

```html
<ul class="terms-tags">
    {{- $type := .Type }}
    {{- range $key, $value := .Data.Terms.Alphabetical }}
    {{- $name := .Name }}
    {{- $count := .Count }}
    {{- with site.GetPage (printf "/%s/%s" $type $name) }}
    <li>
        <a data-weight="{{$count}}" href="{{ .Permalink }}">{{ .Name }} <sup><strong><sup>{{ $count }}</sup></strong></sup> </a>
    </li>
    {{- end }}
    {{- end }}
</ul>
```

## 字体大小按权重显示

`assets/css/common/terms.css`中设计：

```css
.terms-tags a[data-weight="1"] { --size: 1; }
.terms-tags a[data-weight="2"] { --size: 2; }
.terms-tags a[data-weight="3"] { --size: 3; }
.terms-tags a[data-weight="4"] { --size: 4; }
.terms-tags a[data-weight="5"] { --size: 5; }
.terms-tags a[data-weight="6"] { --size: 6; }
.terms-tags a[data-weight="7"] { --size: 7; }
.terms-tags a[data-weight="8"] { --size: 8; }
.terms-tags a[data-weight="9"] { --size: 9; }

.terms-tags a {
    --size: 4;
    font-size: calc(var(--size) * 0.25rem + 1rem);

    /* font-size: 1.4rem; */
    display: block;
    padding: 3px 10px;
    background: var(--tertiary);
    border-radius: 6px;
    transition: transform 0.1s;
}

```

## css伪随机字体色彩

`assets/css/common/terms.css`中

```css
.terms-tags li:nth-child(2n+1) a {color:#f7a400;}
.terms-tags li:nth-child(3n+1) a {color:#3a9efd;}
.terms-tags li:nth-child(4n+1) a {color:#3e4491;}
.terms-tags li:nth-child(5n+1) a {color:#3e4491;}
.terms-tags li:nth-child(6n+1) a {color:#3e4491;}
```

当然了也可以在js里搓qwq，可以参考[这篇](/posts/startsite/basic_setting/#实现随机背景色)












