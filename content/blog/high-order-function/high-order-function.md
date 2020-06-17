---
title: 高阶函数
date: 2020-03-06 12:14:41
tags:
---
## 什么是匿名函数


## 匿名函数的好处

使用匿名函数可以让代码更加简明且易于跟踪

1. 程序的命名空间不会受到那些只使用一次的函数的污染。
2. 函数代码就在调用的那个位置，不用再寻找。
3. 只使用一次的函数往往都是为调用方编写的，这样一来，删除了调用方，被调用的函数也会被删除，管理很方便。

<!-- more -->

## 使用函数作为参数的例子

假设有一个订单处理的算法，它由以下几部分组成：
* 处理订单的每一行记录，并计算总价
* 计算顶单的送货费用
* 验证购买者的信用记录
* 如果成功，那就对该订单进行收费，发送订单确认信息，并将其记录到数据库中。

普通方法是采用硬编码的方式，通过条件语句判断请求的处理类型，但随着可能性数目的增大，这个函数的判断条件会越来越复杂，而且随着每一次条件的变动，这个函数都要变化。
```ruby
def proceess_order(order, user)
    if order
    ...
    elsif order
    ...
    ...
    end
end
```
另一种方法是将这些步骤分别由传递给该算法的一些独立函数进行处理。这样，这个订单处理算法就只需要编写一个简明的通用算法流程。每个主要步骤的细节都是由所传递进来的函数进行处理的。这种算法需要由几个参数：送货计算函数、信用记录验证函数以及订单收费函数。
{% tabs Fourth unique name %}
<!-- tab Ruby -->
```ruby
process_order = ->(order, ship_calc_func, calcute_tax, credit_validate_func) do
    order_lines = order.get_order_lines
    origin = order.get_origin_address
    destination = order.get_destination_address
    delivery_time = order.get_delivery_time
    customer_email = order.get_customer_email

    weight = 0.0
    subtotal = 0.0
    total = 0.0
    shipping = 0.0
    tax = 0.0

    order_lines.each do |e|
        e.weight += e.get_weight
        e.subtotal = total + order_lines.get_price
    end

    shipping = ship_calc_func.call(weight, origin, destination, delivery_time)
    tax = calcute_tax.call(destination, subtotal)
    total = subtotal + shipping + tax
    if credit_validate_func.call(total)
        charge_order_func(total)
        send_confirmation(customer_email, order_lines, shipping, tax, total)
        mark_as_charged(order)
    else
        send_failure_message(customer_email)
        mark_as_failed(order)
    end
end
```
<!-- endtab -->

<!-- tab JavaScript -->
```JavaScript
const processOrder = (order, shipCalcFunc, creditValidateFunc, calculateTax, chargeOrderFunc) => {
    const orderLines = order.getOrderLines();
    const origin = order.getOriginAddress();
    const destination = order.getDestinationAddress();
    const deliverTime = order.getDeliveryTime();
    const customerEmail = order.getCustomerEmail();

    let weight = 0.0;
    let subtotal = 0.0;
    let total = 0.0;
    let tax = 0.0;
    let shipping = 0.0;

    orderLines.forEach(e => {
        e.weight += e.getWeight(); 
        e.subtotal += e.getPrice();
    });
    shipping = shipCalcFunc(weight, origin, destination, deliverTime);
    tax = calculateTax(destination);
    total = subtotal + shipping + tax;
    if (creditValidateFunc(total)) {
        chargeOrderFunc(total);
        sendConfirmation(customerEmail, orderLines, shipping, tax, total);
        markAsCharged(order);
    } else {
        sendFailureMessage(customerEmail);
        markAsFailed(order);
    }
}
```
<!-- endtab -->

<!-- tab Scheme -->
```scheme
(define process-order
    (lambda (order ship-calc-func credit-validate-func charge-order-func) 
        (let 
            (
                (order-lines (get-order-lines order))
                (origin (get-origin-address order))
                (destination (get-destination-address order))
                (delivery-time (get-delivery-time order))
                (customer-email (get-customer-email order))
                (weight 0.0)
                (subtotal 0.0)
                (total 0.0)
                (shipping 0.0)
                (tax 0.0))
            (for-each
                (lambda (order-line)
                    (set! weight (+ weight (get-weight order-line))) 
                    (set! subtotal (+ total (get-price order-line)))) 
                order-lines)
            (set! shipping (ship-calc-func weight origin destination delivery-time))
            (set! tax (calculate-tax destination subtital))
            (set! total (+ subtotal shipping tax))
            (if (credit-validate-func total)
                (begin
                    (charge-order-func total)
                    (send-confirmation customer-email order-lines shipping tax total)
                    (mark-as-charged order)) 
                (begin
                    (send-failure-message customer-email)
                    (mark-as-failed order))))))
```
<!-- endtab -->
{% endtabs %}