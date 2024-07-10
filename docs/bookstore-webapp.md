### **CSS**
```css title="css.xml" linenums="1"
  #app {
    padding-top: 90px;
}

.disabled {
    pointer-events: none;
    cursor: default;
    opacity: 0.5;
}
```
### **Imagen**
  
![GeoLabs BookStore](./files/books.png "GeoLabs BookStore")

### **JS**
- **Books.js**
```js title="books.js" linenums="1"
document.addEventListener('alpine:init', () => {
    Alpine.data('initData', (pageNo) => ({
        pageNo: pageNo,
        books: {
            data: []
        },
        init() {
            this.loadBooks(this.pageNo);
            updateCartItemCount();
        },
        loadBooks(pageNo) {
            $.getJSON("/api/books?page="+pageNo, (resp)=> {
                console.log("Books Resp:", resp)
                this.books = resp;
            });
        },
        addToCart(book) {
            addProductToCart(book)
        }
    }))
});
```

- **Cart.js**
```js title="cart.js" linenums="1"
document.addEventListener('alpine:init', () => {
    Alpine.data('initData', () => ({
        cart: { items: [], totalAmount: 0 },
        user: {
            firstName: "Geovanny",
            lastName: "Mendoza",
            fullName: "Geovanny Manuel Mendoza Gonzalez",
            email: "geovanny@gmail.com",
            phone: "999999999999"
        },
        returnUrl: window.location.origin + "/checkout/paypal/capture",
        init() {
            updateCartItemCount();
            this.loadCart();
            this.cart.totalAmount = getCartTotal();
        },
        loadCart() {
            this.cart = getCart()
        },
        updateCart() {
            this.loadCart();
            this.cart.totalAmount = getCartTotal();
            updateCartItemCount();
        },
        removeItem(id) {
            removeItemFromCart(id);
            this.updateCart();
        },
        removeCart() {
            deleteCart();
        },
        createOrder() {
            let createOrderRequest = {
                customerId: 1,
                bookIds: this.cart.items.map(item => Number(item.id))
            };
            const url = `/checkout/paypal/create?returnUrl=${encodeURIComponent(this.returnUrl)}`;
            $.ajax({
                url: url,
                type: "POST",
                dataType: "json",
                contentType: "application/json",
                data: JSON.stringify(createOrderRequest),
                success: (resp) => {
                    const approveUrl = resp.approveUrl;
                    if (approveUrl) { // Cambio importante
                        this.removeCart();
                        window.location.href = approveUrl;
                    } else {
                        console.error("approveUrl is not defined in the response");
                        alert("Failed to get the approval URL from PayPal");
                    }
                },
                error: (err) => {
                    console.error("PayPal Checkout Error:", err);
                    alert("PayPal checkout failed");
                }
            });
        }
    }))
});
```

- **CartStore.js**
```js title="cartStore.js" linenums="1"

const BOOKSTORE_STATE_KEY = "BOOKSTORE_STATE";

const getCart = function() {
    let cart = localStorage.getItem(BOOKSTORE_STATE_KEY)
    if (!cart) {
        cart = JSON.stringify({items:[], totalAmount:0 });
        localStorage.setItem(BOOKSTORE_STATE_KEY, cart)
    }
    return JSON.parse(cart)
}

const addProductToCart = function(book) {
    let cart = getCart();
    let cartItem = cart.items.find(itemModel => itemModel.id === book.id);
    if (cartItem) {
        cartItem.quantity = parseInt(cartItem.quantity) + 1;
    } else {
        cart.items.push(Object.assign({}, book, {quantity: 1}));
    }
    localStorage.setItem(BOOKSTORE_STATE_KEY, JSON.stringify(cart));
    updateCartItemCount();
}

function updateCartItemCount() {
    let cart = getCart();
    let count = 0;
    cart.items.forEach(item => {
        count = count + item.quantity;
    });
    $('#cart-item-count').text('(' + count + ')');
}

function getCartTotal() {
    let cart = getCart();
    let totalAmount = 0;
    cart.items.forEach(item => {
        totalAmount = totalAmount + (item.price * item.quantity);
    });
    return totalAmount;
}

const removeItemFromCart = function(id) {
    let cart = getCart();
    const index = cart.items.findIndex(item => item.id === id);
    if (index !== -1) {
        cart.items.splice(index, 1);
        localStorage.setItem(BOOKSTORE_STATE_KEY, JSON.stringify(cart));
        updateCartItemCount();
    }
}

const deleteCart = function() {
    localStorage.removeItem(BOOKSTORE_STATE_KEY)
    updateCartItemCount();
}

function clearCart() {
    localStorage.removeItem(BOOKSTORE_STATE_KEY);
    updateCartItemCount();
}

document.addEventListener('DOMContentLoaded', function() {
    updateCartItemCount();
});
```
- **Order.js**
```js title="order.js" linenums="1"
document.addEventListener('alpine:init', () => {
    Alpine.data('initData', () => ({
        orders: [],
        init() {
            this.loadOrders();
            updateCartItemCount();
        },
        loadOrders() {
            $.getJSON("/api/orders", (data) => {
                this.orders = data.map(order => ({
                    orderId: order.items[0].orderId, 
                    paymentStatus: order.paymentStatus
                }));
            });
        }
    }));
});
```

- **OrderDetails.js**
```js title="orderDetails.js" linenums="1"
document.addEventListener('alpine:init', () => {
    Alpine.data('initData', (orderNumber) => ({
        orderNumber: orderNumber,
        orderDetails: {
            items: [],
            customer: {},
            deliveryAddress: {},
            id: orderNumber,
            totalAmount: 0
        },
        init() {
            updateCartItemCount();
            this.getOrderDetails(this.orderNumber);
        },
        getOrderDetails(orderNumber) {
            $.getJSON(`/api/orders/${orderNumber}`, (data) => {
                this.orderDetails = data;
                this.orderDetails.id = orderNumber;
                this.orderDetails.totalAmount = data.total;
            });
        },
        updateItemDetails(orderId, itemId) {
            $.getJSON(`/api/orders/${orderId}/items/${itemId}`, (data) => {
                let itemIndex = this.orderDetails.items.findIndex(item => item.id === itemId);
                if(itemIndex !== -1) {
                    this.orderDetails.items[itemIndex] = data;
                    this.orderDetails.items[itemIndex].imageUrl = data.imageUrl;
                    this.$nextTick(() => {
                        this.$refresh();
                    });
                }
            });
        },
        downloadAndUpdateItemDetails(orderId, itemId) {
            const downloadLink = `/orders/${orderId}/items/${itemId}/book/download`;
            window.open(downloadLink, '_blank');
            setTimeout(() => {
                this.updateItemDetails(orderId, itemId);
            }, 1000);
        }
    }));
});
```

- **Confirmation.js**
```js title="confirmation.js" linenums="1"
document.addEventListener('DOMContentLoaded', function() {
    var completedElement = document.getElementById('paymentStatus');
    if (completedElement) {
        var completed = completedElement.getAttribute('data-completed') === 'true';
        clearCart();
        if (completed) {
            localStorage.setItem('paymentCompleted', 'true');
        }
    }
});
```

### **Templates**
- **Layout**
```html title="layout.html" linenums="1"
<!DOCTYPE html>
<html lang="en"
      xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout">
<head>
    <meta charset="utf-8"/>
    <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
    <title>BookStore</title>
    <meta content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no" name="viewport"/>
    <link rel="stylesheet" href="/webjars/bootstrap/5.3.3/css/bootstrap.css">
    <link rel="stylesheet" href="/webjars/font-awesome/6.5.1/css/all.css">
    <link rel="stylesheet" href="/css/styles.css">
</head>
<body>
<main>
    <nav class="navbar fixed-top navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" href="#" th:href="@{/}">
                <img src="/images/books.png" alt="Books Logo" width="40" height="40"> BookStore
            </a>
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse"
                    data-bs-target="#navbarSupportedContent"
                    aria-controls="navbarSupportedContent" aria-expanded="false"
                    aria-label="Toggle navigation">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id="navbarSupportedContent">
                <ul class="navbar-nav ms-auto mb-2 mb-lg-0">
                    <li class="nav-item">
                        <a class="nav-link" href="/orders" th:href="@{/orders}">
                            Orders
                        </a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="/cart" th:href="@{/cart}">
                            Cart <span id="cart-item-count">(0)</span>
                        </a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>

    <div id="app" class="container">
        <div layout:fragment="content">
            <!-- Your Page Content Here -->
        </div>
    </div>
</main>

<script src="/webjars/jquery/3.7.1/jquery.js"></script>
<script src="/webjars/bootstrap/5.3.3/js/bootstrap.bundle.js"></script>
<script defer src="/webjars/alpinejs/3.13.5/dist/cdn.min.js"></script>
<script src="/js/cartStore.js"></script> 
<div layout:fragment="pageScripts">
</div>
</body>
</html>
```
- **Pagination**
```html title="pagination.html" linenums="1"
<div th:fragment="pagination">
    <template x-if="books.totalPages > 1">
        <nav aria-label="Page navigation" >
            <ul class="pagination pagination justify-content-center">
                <li class="page-item" :class="{ 'disabled': books.isFirst }">
                    <a class="page-link" href="#" th:href="${'/books?page=1'}">First</a>
                </li>
                <li class="page-item" :class="{ 'disabled': !books.hasPrevious }">
                    <a class="page-link" href="#"
                       :href="'/books?page=' + (books.pageNumber-1)">Previous</a>
                </li>
                <li class="page-item" :class="{ 'disabled': !books.hasNext }">
                    <a class="page-link" href="#"
                       :href="'/books?page=' + (books.pageNumber+1)">Next</a>
                </li>
                <li class="page-item" :class="{ 'disabled': books.isLast }">
                    <a class="page-link" href="#"
                       :href="'/books?page=' + (books.totalPages)">Last</a>
                </li>
            </ul>
        </nav>
    </template>
</div>
```
- **Books**
```html title="books.html" linenums="1"
<!DOCTYPE html>
<html lang="en"
      xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorate="~{layout}">
<body>
<div layout:fragment="content">
    <div th:x-data="|initData('${pageNo}')|">
        <div th:replace="~{fragments/pagination :: pagination}"> </div>
        <div class="row row-cols-1 row-cols-md-5">
            <template x-for="book in books.data">
                <div class="col mb-3">
                    <div class="card h-100 book">

                        <img :src="'/api/files/' + book.imageUrl"
                                class="card-img-top"
                                height="300" width="200"
                        />
                        <div class="card-body">
                            <h5 class="card-title"
                                data-toggle="tooltip"
                                data-placement="top"
                                x-text="book.name">book.name</h5>
                            <p class="card-text"
                               data-toggle="tooltip"
                               data-placement="top"
                               x-text="'Price: $'+book.price"
                            >book.price</p>
                        </div>
                        <div class="card-footer" style="background: transparent; border-top: 0;">
                            <div class="d-grid gap-2">
                                <button class="btn btn-primary" @click="addToCart(book)">
                                    Add to Cart
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            </template>
        </div>
    </div>
</div>
<div layout:fragment="pageScripts">
    <script src="/js/books.js"></script>
</div>
</body>
</html>
```
- **Cart**
```html title="cart.html" linenums="1"
<!DOCTYPE html>
<html lang="en"
      xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorate="~{layout}">
<body>
<div layout:fragment="content">
    <div x-data="initData()">
        <div class="container">
            <div class="row">
                <div class="col-md-8">
                    <h2>Ítems del carrito</h2>
                    <div x-show="cart.items.length == 0">
                        <h3>Tu carrito está vacío. <a href="/">Seguir comprando</a></h3>
                    </div>
                    <div x-show="cart.items.length > 0">
                        <div class="list-group">
                            <template x-for="item in cart.items" :key="item.code">
                                <div class="list-group-item d-flex justify-content-between align-items-center">
                                    <div class="d-flex align-items-center">
                                        <img :src="'/api/files/' + item.imageUrl" alt="book image" width="50" height="50" class="mr-3">
                                        <div>
                                            <h5 x-text="item.name" class="mb-1"></h5>
                                            <small x-text="'$' + item.price"></small>
                                        </div>
                                    </div>
                                    <div>
                                        <button class="btn btn-danger btn-sm" @click="removeItem(item.id)">Eliminar</button>
                                    </div>
                                </div>
                            </template>
                        </div>
                    </div>
                </div>
                <div class="col-md-4">
                    <h2>Resumen</h2>
                    <div class="card">
                        <div class="card-body">
                            <h4>Total</h4>
                            <h3 x-text="'$' + cart.totalAmount" class="text-danger"></h3>
                            <form @submit.prevent="createOrder">
                                <button type="submit" class="btn btn-primary btn-block" :disabled="cart.items.length === 0">Pagar con PayPal</button>
                            </form>
                            <a href="/" class="btn btn-secondary btn-block mt-2">Seguir comprando</a>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
<div layout:fragment="pageScripts">
    <script src="https://cdn.jsdelivr.net/npm/alpinejs@2.x.x/dist/alpine.min.js" defer></script>
    <script src="/js/cart.js"></script>
</div>
</body>
</html>
```
- **Order**
```html title="order.html" linenums="1"
<!DOCTYPE html>
<html lang="en"
      xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorate="~{layout}" xmlns:x-bind="http://www.w3.org/1999/xhtml">
<body>
<div layout:fragment="content">
    <div x-data="initData()">
        <h2>All Orders</h2>
        <hr/>
        <div class="pb-3">
            <table class="table">
                <thead>
                <tr>
                    <th scope="col">Order ID</th>
                    <th scope="col">Status</th>
                </tr>
                </thead>
                <tbody>
                <template x-for="order in orders">
                    <tr>
                        <td><a x-bind:href="'/orders/'+order.orderId" x-text="order.orderId">OrderId</a></td>
                        <td x-text="order.paymentStatus">paymentStatus</td>
                    </tr>
                </template>
                </tbody>
            </table>
        </div>
    </div>
</div>
<div layout:fragment="pageScripts">
    <script src="/js/orders.js"></script>
</div>
</body>
</html>
```
- **Order Details**
```html title="order_details.html" linenums="1"
<!DOCTYPE html>
<html lang="en"
      xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorate="~{layout}" xmlns:x-bind="http://www.w3.org/1999/xhtml" xmlns:x-on="http://www.w3.org/1999/xhtml">
<body>
<div layout:fragment="content">
    <div th:x-data="|initData('${orderNumber}')|">
        <h2>Detalles de la venta</h2>
        <div class="row">
            <div class="col-md-8">
                <h3>Ítems de la venta</h3>
                <div class="list-group">
                    <template x-for="item in orderDetails.items" :key="item.id">
                        <div class="list-group-item d-flex justify-content-between align-items-center">
                            <div class="row">
                                <div class="col-md-4">
                                    <img x-bind:src="'/api/files/' + item.imageUrl"
                                         class="card-img-top"
                                         alt="Imagen del libro"
                                         style="width: 100px; height: auto;"
                                    />
                                </div>
                                <div class="col-md-8">
                                    <h5 x-text="item.bookName" class="mb-1"></h5>
                                    <p x-text="'$' + item.price"></p>
                                    <p>Descargas Disponibles: <span x-text="item.downloadsAvailable"></span></p>
                                </div>
                            </div>
                            <div>
                                <a x-bind:href="`/orders/${orderDetails.id}/items/${item.id}/book/download`"
                                   class="btn btn-danger btn-sm"
                                   x-on:click.prevent="downloadAndUpdateItemDetails(orderDetails.id, item.id)"
                                   :class="{ 'disabled': item.downloadsAvailable === 0 }">
                                    <i class="fa fa-download"></i> Descargar PDF
                                </a>
                            </div>
                        </div>
                    </template>
                </div>
            </div>
            <div class="col-md-4">
                <h3>Resumen</h3>
                <div class="card">
                    <div class="card-body">
                        <h4>Total</h4>
                        <h3 x-text="'$' + orderDetails.totalAmount" class="text-danger"></h3>
                        <a href="/" class="btn btn-secondary btn-block mt-2">Seguir comprando</a>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
<div layout:fragment="pageScripts">
    <script src="/js/orderDetails.js"></script>
    <script src="/js/cartStore.js"></script>
</div>
</body>
</html>

```
- **Confirmation**
```html title="confirmation.html" linenums="1"
<!DOCTYPE html>
<html lang="en"
      xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorate="~{layout}">
<body>
<div layout:fragment="content">
    <div class="container">
        <h2>Confirmación de pedido</h2>
        <div th:if="${completed}">
            <p>El pago se ha realizado correctamente!</p>
            <p>Order ID: <span th:text="${orderId}"></span></p>
            <a th:href="@{/orders/{id}(id=${orderId})}" class="btn btn-primary">Ver pedido</a>
        </div>
        <div th:if="${!completed}">
            <p>El pago no se ha realizado correctamente. Por favor, inténtelo de nuevo.</p>
            <a href="/" class="btn btn-secondary">Volver a la página principal</a>
        </div>
    </div>
</div>
<div id="paymentStatus" data-completed="${completed}" style="display:none;"></div>
<script src="/js/cartStore.js"></script>
<script src="/js/confirmation.js"></script>
</body>
</html>
```

### **Conclusión**