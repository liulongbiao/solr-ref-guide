# 处理货币和汇率

`currency` 字段类型给 Solr/Lucene 提供了包含查询时货币转换和汇率在内的金融值。
它支持以下特性：

* 点查询
* 范围查询
* 函数范围查询
* 排序
* 根据货币编码或符号的货币解析
* 对称和非对称汇率(非对称汇率对存在货币转换税率的场景很有用)

## 配置货币

`currency` 字段类型定义在 `schema.xml` 中。
下面是该类型的默认配置：

```xml
<fieldType name="currency" class="solr.CurrencyField" precisionStep="8"
  defaultCurrency="USD" currencyConfig="currency.xml" />
```

本例中，我们定义了名称和字段类型的类，并且定义了 `defaultCurrency` 为 `USD`，及美元。
我们也定义了一个 `currencyConfig` 来使用一个称为 `currency.xml` 的文件。
它是一个包含我们的默认货币和其他货币之间的汇率的文件。
还有一个替代实现允许周期地下载货币数据。详见下文 [汇率](#exchange-rate)

在索引时，金钱字段可以原生的货币进行索引。
如，一个欧洲电商网站上的某个产品，将其价格字段索引为 `1000,EUR` 能正确地索引。
其价格和货币之间应该有一个逗号隔开，且价格必须以浮点值编码 (小数点)。

在查询处理中，范围和点查询都支持。

## <a name="exchange-rate"></a>汇率

你可以通过指定一个提供商来配置汇率。
原生的支持两种类型的提供商： `FileExchangeRateProvider` 或 `OpenExchangeRatesOrgProvider`。

### FileExchangeRateProvider

这个提供商需要你提供一个汇率文件。
它是默认的，即使用它，你只需要指定文件路径和名称作为该类型定义中 `currencyConfig` 的值即可。

在 Solr 中包含了一个示例 `currency.xml`，可在 `schema.xml` 文件相同的目录下找到。
以下是该文件的小片段：

```xml
<currencyConfig version="1.0">
  <rates>
    <!-- Updated from http://www.exchangerate.com/ at 2011-09-27 -->
    <rate from="USD" to="ARS" rate="4.333871" comment="ARGENTINA Peso" />
    <rate from="USD" to="AUD" rate="1.025768" comment="AUSTRALIA Dollar" />
    <rate from="USD" to="EUR" rate="0.743676" comment="European Euro" />
    <rate from="USD" to="CAD" rate="1.030815" comment="CANADA Dollar" />
    <!-- Cross-rates for some common currencies -->
    <rate from="EUR" to="GBP" rate="0.869914" />
    <rate from="EUR" to="NOK" rate="7.800095" />
    <rate from="GBP" to="NOK" rate="8.966508" />

    <!-- Asymmetrical rates -->
    <rate from="EUR" to="USD" rate="0.5" />
  </rates>
</currencyConfig>
```

### OpenExchangeRatesOrgProvider

你可以配置 Solr 从 http://www.OpenExchangeRates.Org 下载汇率，
它每个小时都会更新美元和 158 种货币的汇率。这些汇率只是对称的。

这里，你需要在字段类型的定义中指定 `providerClass`。如下例：

```xml
<fieldType name="currency" class="solr.CurrencyField" precisionStep="8"
  providerClass="solr.OpenExchangeRatesOrgProvider"
  refreshInterval="60"
  ratesFileLocation="http://www.openexchangerates.org/api/latest.json?app_id=yourPersonalAppIdKey"/>
```

其中 `refreshInterval` 是分钟，因此上述示例将每 60 分钟下载一次最新的汇率。
其刷新间隔可以加快，而不能减慢。
