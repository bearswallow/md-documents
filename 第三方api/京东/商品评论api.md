# 获取商品评论

GET https://sclub.jd.com/comment/productPageComments.action?callback=fetchJSON_comment98vv220&productId=3760446&score=0&sortType=5&page=1&pageSize=10&isShadowSku=0&rid=0&fold=1

## 查询参数说明

- callback=fetchJSON_comment98vv220：这个是说明获取的数据格式，且用于解决jsonp的跨域问题。
- productId=376044：产品id，产品html页的url中包含产品id，这个产品的html页url为 <https://item.jd.com/3760446.html>
- score=0：评论类型。0-所有；3-好评；5-追评；7-视频晒单。
- sortType=5：排序类型。目前都是5，按时间倒序排序。
- page=0：页码，从0开始，目前不能超过100（不含100）。
- pageSize=10：每页条数，目前服务器固定每次返回10条。
- rid=0：评论标签id，0表示全部。可以取返回数据中评论标签中的 `rid` 值作为该查询参数的值。

其它的查询参数给固定值即可。

## 返回数据说明

返回数据是jsonp格式的：`fetchJSON_comment98vv220(json数据)`

这里的 `fetchJSON_comment98vv220` 即为查询参数 `callback` 中的值。所以拿到返回数据后首先需要将 json数据 字符串解析出来。

### json数据重要属性

```json
{
    // 产品评论统计信息
    "productCommentSummary": {
        // 好评数
    	"goodCount": 130000,
        // 追评数
    	"afterCount": 400,
        // 视频晒单
    	"videoCount": 200,
        // 评论数量
    	"commentCount": 140000
    },
    // 热门评论标签统计列表
    "hotCommentTagStatistics"：[
    	{
			// 标签名称
            "name": "颜色漂亮",
            // 标签id
            "rid": "8664f0d55c683531",
            // 含该标签的商品数量
            "count": 273,
            // 是否允许筛选
            "canBeFiltered": true
    	}
    ],
    // 评论列表
    "comments": [
        {
            // 评论内容
            "content": "Dior这款567很好的，缪斯玫红色，没有什么味道。老款叫Darling，新款就是现在这个了，应该很好看的，端庄不艳俗，送妹妹的礼物，她很喜欢！用着漂亮迷人，心情好。物流配送一直如此快速！相信京东自营的品质！保持住噢！\n好评！",
            // 创建时间
            "creationTime": "2017-08-08 00:44:24",
            // 是否置顶
            "isTop": false,
            // 客服回复列表
            "replies": [
                {
                    // 回复内容
                    "content": "您好，我们可以保证京东所售商品都是正品/真品，产品都有完整的质检证明和资质，质量都是有严格的把控的，产品都是没有任何问题的，请您放心！\n",
                    // 回复时间
                    "creationTime": "2019-04-24 14:41:40"
                }
            ],
            // 用户回复数量
            "replyCount": 3,
            // 客服回复数量
            "replyCount2": 0,
            // 评论分数
            "score": 5,
            // 标题
            "title": "",
            // 点赞数
            "usefulVoteCount": 31,
            "images": [
                {
                    // 缩略图url，会有京东水印
                    "imgUrl": "//img30.360buyimg.com/n0/s128x96_jfs/t6862/58/2049986360/61139/872425f4/598898e7Ne0d2d265.jpg",
                    // 图片标题
                    "imgTitle": ""
                }
            ],
            // 评论上传视频列表
            "videos": [
                {
                    // 视频标题
                    "videoTitle": "",
                    // 视频url
                    "remark": "https://vod.300hu.com/4c1f7a6atransbjngwcloud1oss/025fb8ae116137034275921921/v.f30.mp4?dockingId=207e0725-b536-414c-90cf-f57dcce53d84&storageSource=3",
                    // 视频秒数
                    "videoLength": 10,
                    // 视频封面图url
                    "mainUrl": "https://img.300hu.com/4c1f7a6atransbjngwcloud1oss/025fb8ae116137034275921921/imageCoverBySnapshot/1542425239_888616832.100_0.jpg",
                    // 视频宽度
                    "videoWidth": 720,
                    // 视频高度
                    "videoHeight": 1280
                }
            ],
            // 订单评论显示内容
            "showOrderComment": {
                // 内容，这里面的<img/>标签对应着评论图片，而且src里的url为原始图片地址，但是不一定每个评论中这部分内容都包含图片信息。所以解析评论图片不能完全依赖这里面的图片信息。
                "content": "Dior这款567很好的，缪斯玫红色，没有什么味道。老款叫Darling，新款就是现在这个了，应该很好看的，端庄不艳俗，送妹妹的礼物，她很喜欢！用着漂亮迷人，心情好。物流配送一直如此快速！相信京东自营的品质！保持住噢！\n好评！<div class='uploadimgdiv'><img class='uploadimg' border='0'  src='http://img30.360buyimg.com/shaidan/jfs/t6862/58/2049986360/61139/872425f4/598898e7Ne0d2d265.jpg' /></div><div class='uploadimgdiv'><img class='uploadimg' border='0'  src='http://img30.360buyimg.com/shaidan/jfs/t5587/129/9658214745/67765/e8acc60/598898e7N8930bd0a.jpg' /></div>"
            },
            // 评论上传图片数量
    		"imageCount": 9,
            // 追评天数
            "afterDays": 6,
            // 追评
            "afterUserComment": {
                "hAfterUserComment": {
                    // 追评内容
                    "content": "挺好的，不会有唇纹，用过的最好的一款口红了"
                }
            },
            // 追评图片列表
            "afterImages": [
                {
                    // 缩略图url，含京东水印
                    "imgUrl": "//img30.360buyimg.com/n0/s128x96_jfs/t29008/90/601388893/184638/150c9f8c/5bf8d843N364d3e25.jpg",
                    // 图片标题
                    "imgTitle": ""
                }
            ]
        }
    ],
	// 允许获取的最大页数
	"maxPage": 100
}
```

评论中的图片url基本上都是缩略图的url，而缩略图大小为 `128x96` 且有京东的水印，所以不能直接使用缩略图，目前发现一个获取该缩略图的另一尺寸 `616x405` 图片的url。它们分别如下

- `128x96` 缩略图url：http://img30.360buyimg.com/n0/s128x96_jfs/t29008/90/601388893/184638/150c9f8c/5bf8d843N364d3e25.jpg
- `616x405` 缩略图url：http://img30.360buyimg.com/shaidan/s616x405_jfs/t29008/90/601388893/184638/150c9f8c/5bf8d843N364d3e25.jpg

只需要将 `128x96` 缩略图url中的 `n0/s128x96` 替换成 `shaidan/s616x405` 就能得到 `616x405` 缩略图的url。

### json完整属性说明

#### 产品评论统计信息

```json
{
    // 好评率显示值
    "goodRateShow": 98,
    // 差评率显示值
    "poorRateShow": 2,
    "poorCountStr": "800+",
    // 平均分
    "averageScore": 5,
    "generalCountStr": "500+",
    "oneYear": 0,
    "showCount": 6300,
    "showCountStr": "6300+",
    // 好评数
    "goodCount": 130000,
    // 中评率
    "generalRate": 0.004,
    // 中评数
    "generalCount": 500,
    "skuId": 3760446,
    "goodCountStr": "13万+",
    // 差评率
    "poorRate": 0.016,
    "sensitiveBook": 0,
    // 追评数
    "afterCount": 400,
    "goodRateStyle": 147,
    // 差评数
    "poorCount": 800,
    "skuIds": null,
    // 视频晒单
    "videoCount": 200,
    "poorRateStyle": 2,
    "generalRateStyle": 1,
    "commentCountStr": "14万+",
    // 评论数量
    "commentCount": 140000,
    // 商品id
    "productId": 3760446,
    "videoCountStr": "200+",
    "afterCountStr": "400+",
    "defaultGoodCount": 110000,
    //好评率
    "goodRate": 0.98,
    // 中评率显示值
    "generalRateShow": 0,
    "defaultGoodCountStr": "11万+"
}
```

#### 热门评论标签统计

```json
{
    "id": "8664f0d55c683531",
    // 标签名称
    "name": "颜色漂亮",
    // 标签id
    "rid": "8664f0d55c683531",
    // 含该标签的商品数量
    "count": 273,
    "type": 4,
    // 是否允许筛选
    "canBeFiltered": true,
    "stand": 1,
    "ckeKeyWordBury": "eid=100^^tagid=8664f0d55c683531^^pid=20006^^sku=3760446^^sversion=1000^^token=1a2f10cf3bf18d72",
    "flag": "false"
}
```

#### 评论

```json
{
    // 评论id
    "id": 10681890470,
    "topped": 0,
    "guid": "544a5a1f-b37c-43a7-bfa3-d2299958d4f4",
    // 评论内容
    "content": "Dior这款567很好的，缪斯玫红色，没有什么味道。老款叫Darling，新款就是现在这个了，应该很好看的，端庄不艳俗，送妹妹的礼物，她很喜欢！用着漂亮迷人，心情好。物流配送一直如此快速！相信京东自营的品质！保持住噢！\n好评！",
    // 创建时间
    "creationTime": "2017-08-08 00:44:24",
    // 是否置顶
    "isTop": false,
    // 关联对象id
    "referenceId": "2531316",
    // 关联对象图
    "referenceImage": "jfs/t17173/11/987480935/93070/fe9eab1e/5ab47b47N56a9ce8a.jpg",
    // 关联对象名称
    "referenceName": "迪奥（Dior）烈艳蓝金唇膏口红3.5g 080(滋润保湿 妆感舒悦 微笑红色)",
    // 关联时间
    "referenceTime": "2017-08-07 00:18:34",
    // 关联对象类型，这里是产品
    "referenceType": "Product",
    // 关联对象类型id
    "referenceTypeId": 0,
    // 商品一级分类
    "firstCategory": 1316,
    // 商品二级分类
    "secondCategory": 1387,
    // 商品三级分类
    "thirdCategory": 1425,
    // 客服回复列表
    "replies": [
        {
            "commentId": "12684651412",
            // 回复内容
            "content": "您好，我们可以保证京东所售商品都是正品/真品，产品都有完整的质检证明和资质，质量都是有严格的把控的，产品都是没有任何问题的，请您放心！\n",
            // 回复时间
            "creationTime": "2019-04-24 14:41:40",
            "creationTimeString": "2019-04-24 14:41:40",
            "id": 455282170,
            "guid": "455282170",
            "isDelete": false,
            "parentId": "0",
            "targetId": "null",
            "pin": "京东美妆专员",
            "userImage": "misc.360buyimg.com/user/myjd-2015/css/i/peisong.jpg",
            "userLevelId": "50",
            "userProvince": "",
            // 回复人
            "nickname": "京东美妆专员",
            "userClient": 102,
            "updateTime": "2019-04-24 14:41:42",
            "plusAvailable": 0,
            "replyCount": 0,
            "aesPin": "Lg_0FZExQ5x5ztFX3wojFHBjrmdSXezVn5XEBeACwhpudbG8BK8hXBthAgZyXi8RgybkHOFXp3cnTnEso5sm3g",
            "replyList": [],
            "praiseCount": 0,
            "userLevelName": "注册会员",
            "userClientShow": "",
            "isMobile": false
        }
    ],
    // 用户回复数量
    "replyCount": 3,
    // 客服回复数量
    "replyCount2": 1,
    // 评论分数
    "score": 5,
    // 状态
    "status": 1,
    // 标题
    "title": "",
    // 点赞数
    "usefulVoteCount": 31,
    // 觉得没用的数量
    "uselessVoteCount": 0,
    // 用户头像url
    "userImage": "storage.360buyimg.com/i.imageUpload/b7e3b5a4b0d7c2b431353438353932343930333131_sma.jpg",
    // 用户头像url
    "userImageUrl": "storage.360buyimg.com/i.imageUpload/b7e3b5a4b0d7c2b431353438353932343930333131_sma.jpg",
    // 用户等级id
    "userLevelId": "105",
    // 用户所在省份
    "userProvince": "",
    "viewCount": 0,
    "orderId": 0,
    "isReplyGrade": false,
    // 用户昵称
    "nickname": "山***彡",
    // 用户客户端
    "userClient": 2,
    // 评论上传的图片列表
    "images": [
        {
            // 评论id
            "id": 384299518,
            "associateId": 243662028,
            "productId": 0,
            // 缩略图url，会有京东水印
            "imgUrl": "//img30.360buyimg.com/n0/s128x96_jfs/t6862/58/2049986360/61139/872425f4/598898e7Ne0d2d265.jpg",
            "available": 1,
            "pin": "",
            "dealt": 0,
            // 图片标题
            "imgTitle": "",
            "isMain": 0,
            "jShow": 0
        }
    ],
    // 评论上传视频列表
    "videos": [
        {
            "id": 712481171,
            "associateId": 456025913,
            "productId": 3277093,
            // 视频标题
            "videoTitle": "",
            // 视频url
            "remark": "https://vod.300hu.com/4c1f7a6atransbjngwcloud1oss/025fb8ae116137034275921921/v.f30.mp4?dockingId=207e0725-b536-414c-90cf-f57dcce53d84&storageSource=3",
            "yn": 0,
            "dealt": 0,
            "isMain": 0,
            // 视频秒数
            "videoLength": 10,
            "pin": "",
            "videoUrl": "44764920",
            // 视频封面图url
            "mainUrl": "https://img.300hu.com/4c1f7a6atransbjngwcloud1oss/025fb8ae116137034275921921/imageCoverBySnapshot/1542425239_888616832.100_0.jpg",
            "available": 1,
            // 视频宽度
            "videoWidth": 720,
            // 视频高度
            "videoHeight": 1280,
            "jShow": 0
        }
    ],
    // 评论显示内容
    "showOrderComment": {
        "id": 243662028,
        "guid": "72704f8e-bc02-4908-953a-0b06ea7c0ca3",
        // 内容，这里面的<img/>标签对应着评论图片，而且src里的url为原始图片地址，但是不一定每个评论中这部分内容都包含图片信息。
        "content": "Dior这款567很好的，缪斯玫红色，没有什么味道。老款叫Darling，新款就是现在这个了，应该很好看的，端庄不艳俗，送妹妹的礼物，她很喜欢！用着漂亮迷人，心情好。物流配送一直如此快速！相信京东自营的品质！保持住噢！\n好评！<div class='uploadimgdiv'><img class='uploadimg' border='0'  src='http://img30.360buyimg.com/shaidan/jfs/t6862/58/2049986360/61139/872425f4/598898e7Ne0d2d265.jpg' /></div><div class='uploadimgdiv'><img class='uploadimg' border='0'  src='http://img30.360buyimg.com/shaidan/jfs/t5587/129/9658214745/67765/e8acc60/598898e7N8930bd0a.jpg' /></div><div class='uploadimgdiv'><img class='uploadimg' border='0'  src='http://img30.360buyimg.com/shaidan/jfs/t6850/35/2087275047/81798/b4868dad/598898e7N13b83be5.jpg' /></div><div class='uploadimgdiv'><img class='uploadimg' border='0'  src='http://img30.360buyimg.com/shaidan/jfs/t5797/117/7652553842/86774/a1098ef7/598898e7Na468b698.jpg' /></div><div class='uploadimgdiv'><img class='uploadimg' border='0'  src='http://img30.360buyimg.com/shaidan/jfs/t7144/289/1177077372/78845/55a91364/598898d9N78d12fbf.jpg' /></div><div class='uploadimgdiv'><img class='uploadimg' border='0'  src='http://img30.360buyimg.com/shaidan/jfs/t5941/3/8452723320/68096/24ea8c10/598898e7N23ea0e32.jpg' /></div><div class='uploadimgdiv'><img class='uploadimg' border='0'  src='http://img30.360buyimg.com/shaidan/jfs/t5584/336/9644490378/81237/b172d53/598898e7Nd09fe361.jpg' /></div><div class='uploadimgdiv'><img class='uploadimg' border='0'  src='http://img30.360buyimg.com/shaidan/jfs/t5998/285/8343843636/77804/c8464c58/598898e7Nfdfaa5ce.jpg' /></div><div class='uploadimgdiv'><img class='uploadimg' border='0'  src='http://img30.360buyimg.com/shaidan/jfs/t7075/286/1177532775/75403/693625e4/598898e7N17fb7f0a.jpg' /></div>",
        "creationTime": "2017-08-08 00:44:24",
        "isTop": false,
        "referenceId": "2531316",
        "referenceType": "Order",
        "referenceTypeId": 0,
        "firstCategory": 0,
        "secondCategory": 0,
        "thirdCategory": 0,
        "replyCount": 0,
        "replyCount2": 0,
        "score": 0,
        "status": 1,
        "usefulVoteCount": 0,
        "uselessVoteCount": 0,
        "userProvince": "",
        "viewCount": 0,
        "orderId": 0,
        "isReplyGrade": false,
        "userClient": 2,
        "isDeal": 1,
        "integral": -20,
        "userImgFlag": 0,
        "anonymousFlag": 1,
        "mobileVersion": "",
        "recommend": false,
        // 用户客户端显示内容
        "userClientShow": "来自京东iPhone客户端",
        // 是否移动端
        "isMobile": true,
        "userLevelColor": "#666666"
    },
    "mergeOrderStatus": 2,
    "discussionId": 243662028,
    "productColor": "567#缪斯玫红",
    "productSize": "",
    // 评论上传图片数量
    "imageCount": 9,
    "integral": -20,
    "userImgFlag": 1,
    // 是否匿名
    "anonymousFlag": 1,
    // 用户等级名称
    "userLevelName": "PLUS会员",
    "plusAvailable": 201,
    "productSales": [
        {
            "dim": 3,
            "saleName": "567#缪斯玫红",
            "saleValue": ""
        }
    ],
    "mobileVersion": "",
    "officialsStatus": 0,
    "recommend": true,
    "userClientShow": "来自京东iPhone客户端",
    "isMobile": true,
    "userLevelColor": "#e1a10a",
    "days": 1,
    // 追评天数
    "afterDays": 6,
    // 追评
    "afterUserComment": {
        "tableNames": {},
        "forceRead2Writer": false,
        "id": 38334463,
        "productId": 3277093,
        "orderId": 0,
        "commentId": 12532988042,
        "status": 1,
        "pin": "",
        "clientType": 2,
        "ip": "",
        "anonymous": 1,
        "dealt": 0,
        "created": "2019-03-12 19:07:41",
        "modified": "2019-03-12 19:07:41",
        "hAfterUserComment": {
            "rowKey": "7438334463",
            "id": 38334463,
            // 追评内容
            "content": "挺好的，不会有唇纹，用过的最好的一款口红了",
            "discussionId": 511386308
        }
    },
    // 追评图片列表
    "afterImages": [
        {
            "id": 719730355,
            "associateId": 460664576,
            "productId": 0,
            "imgUrl": "//img30.360buyimg.com/n0/s128x96_jfs/t29008/90/601388893/184638/150c9f8c/5bf8d843N364d3e25.jpg",
            "available": 1,
            "pin": "",
            "dealt": 0,
            "imgTitle": "",
            "isMain": 0,
            "jShow": 0
        },
        {
            "id": 719730356,
            "associateId": 460664576,
            "productId": 0,
            "imgUrl": "//img30.360buyimg.com/n0/s128x96_jfs/t29551/98/615197260/155688/3f7814d4/5bf8d844N8f243426.jpg",
            "available": 1,
            "pin": "",
            "dealt": 0,
            "imgTitle": "",
            "isMain": 0,
            "jShow": 0
        },
        {
            "id": 719730357,
            "associateId": 460664576,
            "productId": 0,
            "imgUrl": "//img30.360buyimg.com/n0/s128x96_jfs/t30403/185/287227773/155433/e3cb2c26/5bf8d844Nfc68c66e.jpg",
            "available": 1,
            "pin": "",
            "dealt": 0,
            "imgTitle": "",
            "isMain": 0,
            "jShow": 0
        }
    ]
}
```

