---
layout: post
title: 'nameko example中marshmallow包介绍'
date: 2017-09-15
author: Pengfei.X
version: 0.1
categories: [nameko, ]
---


直接上代码了，使用marshmallow定义了接口的输入和输出数据格式，服务间调用时，
也是通过这个约定的schema来进行，目前我们json-rpc就是缺乏这种数据格式的定义。


	from marshmallow import Schema, fields


	class CreateOrderDetailSchema(Schema):
	    product_id = fields.Str(required=True)
	    price = fields.Decimal(as_string=True, required=True)
	    quantity = fields.Int(required=True)



orderservice，通过nameko的rpc.decorator注册rpc服务。

	class OrdersService:
	    name = 'orders'

	    db = DatabaseSession(DeclarativeBase)
	    event_dispatcher = EventDispatcher()

	    @rpc
	    def get_order(self, order_id):
	        order = self.db.query(Order).get(order_id)

	        if not order:
	            raise NotFound('Order with id {} not found'.format(order_id))

	        return OrderSchema().dump(order).data

	    @rpc
	    def create_order(self, order_details):
	        order = Order(
	            order_details=[
	                OrderDetail(
	                    product_id=order_detail['product_id'],
	                    price=order_detail['price'],
	                    quantity=order_detail['quantity']
	                )
	                for order_detail in order_details
	            ]
	        )
	        self.db.add(order)
	        self.db.commit()

	        order = OrderSchema().dump(order).data

	        self.event_dispatcher('order_created', {
	            'order': order,
	        })

	        return order



外部可以通过rpc.proxy来调用，注意这里的shema的使用。

	class GatewayService(object):
	    """
	    Service acts as a gateway to other services over http.
	    """

	    name = 'gateway'

	    config = Config()
	    orders_rpc = RpcProxy('orders')
	    products_rpc = RpcProxy('products')

	    @http(
	        "GET", "/products/<string:product_id>",
	        expected_exceptions=ProductNotFound
	    )
	    def get_product(self, request, product_id):
	        """Gets product by `product_id`
	        """
	        product = self.products_rpc.get(product_id)
	        return Response(
	            ProductSchema().dumps(product).data,
	            mimetype='application/json'
	        )