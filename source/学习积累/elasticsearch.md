# 字段

对于object对象，一个字段可以嵌套多个字段

```json
{
	"properties": {
		"age": {
			"type": "integer"
		},
		"name": {
			"properties": {
				"first": {
					"type": "text"
				},
				"last": {
					"type": "text"
				}
			}
		}
	}
}
```



ES支持的数据类型，

1. 字符串类型

   - text ，
     - 适合全文索引，
     - 可以切词，
     - 不用于排序，很少用于聚合，适合长文本
   - keyword
     - 只能通过精准匹配
     - 适合用于过滤、排序、聚合

2. 整数类型。在满足需求的情况下，尽可能选择范围小的数据类型，字段的长度越短，索引和搜索的效率越高。

   1. long
   2. integer
   3. short
   4. byte

3. 浮点类型

   1. double：64位双精度IEEE 754浮点类型
   2. float：32位单精度IEEE 754浮点类型 
   3. half_float：16位半精度IEEE 754浮点类型 
   4. scaled_float ： 缩放类型的的浮点数 

4. 逻辑类型

   1. boolean

5. 日期类型

   1. date日期格式字符串
   2. 时间戳，毫秒级

6. 范围类型。范围类型要求字段值描述的是一个数值、日期或IP地址的范围， 在添加文档时可以使用： gte、gt、lt、lte分别表示 >=、 > 、< 、<= 。

   1. range
      1. integer_range
      2. float_range
      3. long_range
      4. double_range
      5. data_range
      6. ip_range

7. 二进制类型。[二进制](https://so.csdn.net/so/search?q=二进制&spm=1001.2101.3001.7020)字段是指用base64来表示索引中存储的二进制数据，可用来存储二进制形式的数据，例如图像。默认情况下，该类型的字段只存储不索引。二进制类型只支持index_name属性

   1. binary

8. 复合类型

   1. 数组类型 array 
      对象类型 object ：JSON格式对象数据
      嵌套类型 nested 
      地理类型 地理坐标类型 geo_point 
      地理地图 geo_shape 
      特殊类型 IP类型 ip 
      范围类型 completion 
      令牌计数类型 token_count 
      附件类型 attachment 
      抽取类型 percolator 

9. 多数据类型

   1. 有些字段可能会以不同的方式进行检索，如果文档字段只以一种方式编入索引，检索性能就会受到影响。所以针对text和keyword，es专门提供了一个配置字段多数据类型的参数fields，它可以让一个字段同时具备两种数据类型的特征。

      ```json
      PUT articles{
          "mappings"{
              "properties":{
                  "title":{
                      "type":"text",
                      "fields":{
                          "raw":{"type":"keyword"},
                          "length":{
      												"type":"token_count", 
                            	"analyzer":"standard"
                          }
                      }
                  }
              }
          }
      }
      ```

      title字段被设置为text，同时通过fields参数又为该字段添加了两个子字段raw和length，且分别为keyword类型和token_count类型。使用fields设置的子字段，在添加文档时不需要不需要单独设置字段值， 他们与title共享相同的字段值， 只是会以不同的方式处理字段值， 且在查询时不会展现出来。

      ```json
      PUT demo2
      {
          "settings": {
              "number_of_shards": 4,
              "number_of_replicas": 1
          },
          "mappings": {
              "properties": {
                  "id": {
                      "type": "long"
                  },
                  "title": {
                      "type": "text"
                  },
                  "university": {
                      "type": "object",
                      "properties": {
                          "name": {
                              "type": "text"
                          },
                          "foundingYear": {
                              "type": "date"
                          },
                          "address": {
                              "type": "text"
                          }
                      }
                  }
              }
          }
      }
      ```

      

es一个字段可以定义多个分词器

```json

```



拼音分词器

首先配置tokenizer，它的工作是把”2018世界杯”切分成全拼和首拼，一共就2个term。

### tokenizer

```json
"tokenizer" => 
[ 
    // 拼音分词 
    "name_py_tokenizer" => 
        [ "type" => "pinyin",   "keep_none_chinese" => false, // 对非中文不拆分词 
        "keep_full_pinyin" => false, // 关闭: 刘德华 -> liu, de, hua   
        "keep_joined_full_pinyin" => true, // 刘德华 -> liudehua 
        "keep_none_chinese_in_joined_full_pinyin" => true,  // 刘德华2016 -> liudehua2016   
        "keep_first_letter" => true, // 刘德华 -> ldh 
        "keep_none_chinese_in_first_letter" => true, // 刘德华2016 -> ldh2016   
        "none_chinese_pinyin_tokenize" => false, // 没有卵用 
        ]
 ]
```

如果我们要索引”世界杯worldcup”，那么keep_none_chinese=true则会导致把非中文部分拆出1个独立的term：worldcup，这不是我们希望的，我们只需要shijiebeiworldcup或者sjbworldcup，所以这个选项需要设置false。

如果我们要索引”世界杯worldcup”，那么keep_none_chinese=true则会导致把非中文部分拆出1个独立的term：worldcup，这不是我们希望的，我们只需要shijiebeiworldcup或者sjbworldcup，所以这个选项需要设置false。

### filter

接着，需要利用ES内置的edge-ngram filter，把全拼和首字拼按前缀拆term，也就是上面我们列举的大量的前缀term：

Tokenizer 后续深度研究 [分词器对比](https://blog.51cto.com/u_11440114/5100691)

standard 按单词进行分割
letter 按非字符类进行分割
whitespace 按空格进行分割
UAX URL Email 按 standard 分割，但不会分割邮箱和 url
NGram 和 Edge NGram 连词分割
Path Hierachy 

