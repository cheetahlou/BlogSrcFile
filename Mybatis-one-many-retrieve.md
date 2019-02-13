---
title: Mybatis-one-many-retrieve
date: 2019-02-01 23:37:00
tags:
- java
- Mybatis
---

﻿

## 1.需求

有一个需求是这样的，我要查一个订单列表，列表中的一个订单中可能有多个商品，是一对多的关系，然后需要在列表中同时查出来。结果大概是下面这样的：

<!--more-->

![订单列表图](http://img.blog.csdn.net/20171107203945479?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlZXRhaGxvdmVy/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 2.实现

没时间了，直接上xml代码，基本思路是将分页和条件查询放在取一端的id集的操作中，将符合条件的订单id先全部拿出来，再去查这些订单的订单基本信息和多个商品信息，条件查询可以查询订单（一方）和订单商品和卖家姓名（多端）关键字检索。碰到的问题有总数不对、每页实际返回条数不对（Mybatis会将相同的键的拼到同一个result中）、多端有数据填充List时不完整、做分页的时候最后一条只取了多端的一部分，另一部分丢失了。
```
	//select 语句
    <select id="selectOrderListWithCondition" resultMap="mOrders">
        select
        <include refid="mOrdersPkSql"/>
        ,
        muser.id AS Muser_id,
        muser.user_name AS Muser_userName,
        order_goods.id AS OrderGoods_id,
        order_goods.sku_count AS OrderGoods_skuCount,
        order_goods.goods_id AS OrderGoods_goodsId,
        order_goods.sku_id AS OrderGoods_skuId,
        goods.id AS Goods_id,
        goods.goods_name AS Goods_goodsName,
        goods.goods_url AS Goods_goodsUrl,
        sku.id AS Sku_id,
        sku.values_str AS Sku_valuesStr,
        sku.price AS Sku_price,
        sku.inventory AS Sku_inventory,
        sku.if_uniform_spec AS Sku_ifUniformSpec
        from
        orders,
        muser,
        order_goods ,
        goods,
        sku
        WHERE
        orders.id=order_goods.order_id
        AND order_goods.goods_id=goods.id
        AND order_goods.sku_id=sku.id
        AND orders.muser_id=muser.id
        AND orders.id in ( SELECT
        t.orderId
        FROM
        (
        select orders.id orderId
        from
        orders,
        muser,
        order_goods ,
        goods
        WHERE
        orders.id=order_goods.order_id
        AND order_goods.goods_id=goods.id
        AND orders.muser_id=muser.id
        <if test="searchText!=null and searchText!=''">
            AND orders.order_number like concat('%',#{searchText},'%')
        </if>
        <if test="sourceType!=null and sourceType!=''">
            AND orders.source_type=#{sourceType}
        </if>
        <if test="orderType!=null and orderType!=''">
            AND orders.order_type=#{orderType}
        </if>
        <if test="muserId!=null and muserId!=''">
            AND orders.muser_id=#{muserId}
        </if>
        <if test="orderStatus!=null and orderStatus!='' and orderStatus==99 ">
            AND orders.order_status in (4,5)
        </if>
        <if test="orderStatus!=null and orderStatus!='' and orderStatus!=99 ">
            AND orders.order_status = #{orderStatus}
        </if>
        <if test="goodsName!=null and goodsName!=''">
            AND goods.goods_name like concat('%',#{goodsName},'%')
        </if>
        <if test="userName!=null and userName!=''">
            AND muser.user_name like concat('%',#{userName},'%')
        </if>
        GROUP BY orders.id
        ORDER BY orders.gmt_datetime desc
        limit #{startIndex},#{size}
        ) t)
        ORDER BY orders.gmt_datetime desc
    </select>
    
	//统计分页总条数
    <select id="selectOrderListSize" resultType="java.lang.Integer">
        SELECT count(*) FROM
        (SELECT
        orders.id
        FROM
        orders,
        muser,
        order_goods ,
        goods
        WHERE
        orders.id=order_goods.order_id
        AND order_goods.goods_id=goods.id
        AND orders.muser_id=muser.id
        <if test="searchText!=null and searchText!=''">
            AND orders.order_number like concat('%',#{searchText},'%')
        </if>
        <if test="sourceType!=null and sourceType!=''">
            AND orders.source_type=#{sourceType}
        </if>
        <if test="orderType!=null and orderType!=''">
            AND orders.order_type=#{orderType}
        </if>
        <if test="muserId!=null and muserId!=''">
            AND orders.muser_id=#{muserId}
        </if>
        <if test="orderStatus!=null and orderStatus!='' and orderStatus==99 ">
            AND orders.order_status in (4,5)
        </if>
        <if test="orderStatus!=null and orderStatus!='' and orderStatus!=99 ">
            AND orders.order_status = #{orderStatus}
        </if>
        <if test="goodsName!=null and goodsName!=''">
            AND goods.goods_name like concat('%',#{goodsName},'%')
        </if>
        <if test="userName!=null and userName!=''">
            AND muser.user_name like concat('%',#{userName},'%')
        </if>
        GROUP BY orders.id) t
    </select>
```
***
下面是resultMap,用`collection`标签接收list，`association` 接收实体：
```
    <resultMap type="com.rongke.model.Orders" id="mOrders">
        <id property="id" column="Orders_id"/>
        <result property="muserId" column="Orders_muserId"/>
        <result property="addressId" column="Orders_addressId"/>
        <result property="orderNumber" column="Orders_orderNumber"/>
        <result property="outTradeNo" column="Orders_outTradeNo"/>
        <result property="sourceType" column="Orders_sourceType"/>
        <result property="totalPrice" column="Orders_totalPrice"/>
        <result property="returnDuobi" column="Orders_returnDuobi"/>
        <result property="returnYabi" column="Orders_returnYabi"/>
        <result property="cashPayPrice" column="Orders_cashPayPrice"/>
        <result property="allowPayType" column="Orders_allowPayType"/>
        <result property="orderType" column="Orders_orderType"/>
        <result property="orderStatus" column="Orders_orderStatus"/>
        <result property="deliveryMoney" column="Orders_deliveryMoney"/>
        <result property="useCoupon" column="Orders_useCoupon"/>
        <result property="couponVal" column="Orders_couponVal"/>
        <result property="leavingMessage" column="Orders_leavingMessage"/>
        <result property="isLogistics" column="Orders_isLogistics"/>
        <result property="receiverPhone" column="Orders_receiverPhone"/>
        <result property="receiverName" column="Orders_receiverName"/>
        <result property="receiverAddress" column="Orders_receiverAddress"/>
        <result property="logiCom" column="Orders_logiCom"/>
        <result property="logisticsNumber" column="Orders_logisticsNumber"/>
        <result property="payDatetime" column="Orders_payDatetime"/>
        <result property="completeDatetime" column="Orders_completeDatetime"/>
        <result property="payType" column="Orders_payType"/>
        <result property="merchantRemarks" column="Orders_merchantRemarks"/>
        <result property="cusServiceRemarks" column="Orders_cusServiceRemarks"/>
        <result property="gmtDatetime" column="Orders_gmtDatetime"/>
        <result property="uptDatetime" column="Orders_uptDatetime"/>
        <association property="muser" javaType="com.rongke.model.Muser">
            <id property="id" column="Muser_id"/>
            <result property="userName" column="Muser_userName"/>
        </association>
        <collection property="orderGoodsList" ofType="com.rongke.model.OrderGoods" column="OrderGoods_id">
            <result property="id" column="OrderGoods_id"/>
            <result property="goodsId" column="OrderGoods_goodsId"/>
            <result property="skuId" column="OrderGoods_skuId"/>
            <result property="skuCount" column="OrderGoods_skuCount"/>
            <association property="sku" javaType="com.rongke.model.Sku" column="OrderGoods_skuId">
                <id property="id" column="Sku_id"/>
                <result property="price" column="Sku_price"/>
                <result property="valuesStr" column="Sku_valuesStr"/>
                <result property="inventory" column="Sku_inventory"/>
                <result property="ifUniformSpec" column="Sku_ifUniformSpec"/>
            </association>
            <association property="goods" javaType="com.rongke.model.Goods" column="OrderGoods_goodsId">
                <id property="id" column="Goods_id"/>
                <result property="goodsName" column="Goods_goodsName"/>
                <result property="goodsUrl" column="Goods_goodsUrl"/>
            </association>
        </collection>
    </resultMap>
```