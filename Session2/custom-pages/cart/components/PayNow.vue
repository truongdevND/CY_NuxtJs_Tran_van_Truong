<script setup>
import { ref, computed } from "vue"; 
import useCartStore from "~/stores/cartStore";
import notificationStore from "~/stores/notificationStore";
import { addOrder } from "~/service/carts";

const textPhone = ref("");
const textAddress = ref("");
const cartStore = useCartStore();
const useNotificationStore = notificationStore();


const itemOrder = computed(() => {
  if (cartStore.cartList.length === 0) return null;
  return {
    address: textAddress.value,
    phone: textPhone.value,
    cart_item: cartStore.cartList.map((item) => ({
      product_id: item.id,
      quantity: item.quantity,
      price: item.price,
      name: item.name,
    })),
  };
});

const handleSubmit = async () => {
  if (!textPhone.value || !textAddress.value) {
    useNotificationStore.dialog(true);
    useNotificationStore.imageMessage("https://media.istockphoto.com/id/1407160246/vector/danger-triangle-icon.jpg?s=612x612&w=0&k=20&c=BS5mwULONmoEG9qPnpAxjb6zhVzHYBNOYsc7S5vdzYI=");
    useNotificationStore.titleMessage("WARNING!!!!");
    useNotificationStore.mess("Please fill in all required fields");
    return;
  }

  if (!itemOrder.value) {
    useNotificationStore.dialog(true);
    useNotificationStore.imageMessage("https://media.istockphoto.com/id/1407160246/vector/danger-triangle-icon.jpg?s=612x612&w=0&k=20&c=BS5mwULONmoEG9qPnpAxjb6zhVzHYBNOYsc7S5vdzYI=");
    useNotificationStore.titleMessage("Cart is empty");
    useNotificationStore.mess("Please add items to the cart before placing an order");
    return;
  }

  try {
    await addOrder(
      itemOrder.value.address,
      itemOrder.value.phone,
      itemOrder.value.cart_item
    );
    cartStore.clearCart();

    useNotificationStore.dialog(true);
    useNotificationStore.titleMessage("Order Success");
    useNotificationStore.imageMessage("https://static-00.iconduck.com/assets.00/success-icon-512x512-qdg1isa0.png");
    useNotificationStore.mess("Your order has been placed successfully");
  } catch (error) {
    useNotificationStore.dialog(true);
    useNotificationStore.imageMessage("https://media.istockphoto.com/id/1407160246/vector/danger-triangle-icon.jpg?s=612x612&w=0&k=20&c=BS5mwULONmoEG9qPnpAxjb6zhVzHYBNOYsc7S5vdzYI=");
    useNotificationStore.titleMessage("Error");
    useNotificationStore.mess("Failed to place order: " + error.message);
  }
};
</script>


<template>
  <div class="bg-gray-200 p-8 antialiased md:py-16">
    <div class="mx-auto max-w-5xl">
      <h2 class="text-xl font-semibold text-gray-900 sm:text-2xl">Payment</h2>
      <div class="mt-6 sm:mt-8 flex flex-col lg:items-start lg:gap-12">
        <form class="w-full rounded-lg border border-gray-200 bg-white p-4 shadow-sm sm:p-6 lg:max-w-xl lg:p-8">
          <div class="col-span-2 sm:col-span-1">
            <label for="full_name" class="mb-2 block text-sm font-medium text-gray-900">Phone</label>
            <input type="text" v-model="textPhone" placeholder="Phone number ..."
              class="block w-full rounded-lg border border-gray-300 bg-gray-50 p-2.5 text-sm text-gray-900 focus:border-primary-500 focus:ring-primary-500"
              required />
          </div>
          <div>
            <label for="address"
              class="mb-2 mt-4 flex items-center gap-1 text-sm font-medium text-gray-900">Address</label>
            <input type="text" v-model="textAddress" placeholder="Address ..."
              class="block w-full rounded-lg border border-gray-300 bg-gray-50 p-2.5 text-sm text-gray-900 focus:border-primary-500 focus:ring-primary-500"
              required />
          </div>
          <button type="button" @click.prevent="handleSubmit"
            class="mt-8 max-w-full text-sm px-6 py-3 w-full bg-blue-600 hover:bg-blue-700 text-white font-semibold tracking-wide rounded-lg">
            Pay Now
          </button>
          <DialogNotification />
        </form>
        <div class="mt-6 w-full grow sm:mt-8 lg:mt-0">
          <div class="space-y-4 rounded-lg border border-gray-100 bg-gray-50 p-6">
            <div class="flex items-center justify-between gap-4 border-t border-gray-200 pt-2">
              <div class="text-base font-bold text-gray-900">Total</div>
              <dd class="text-base font-bold text-gray-900">
                $ {{ cartStore.totalPrice }}
              </dd>
            </div>
          </div>
          <div class="mt-6 flex items-center justify-center gap-8"></div>
        </div>
      </div>
    </div>
  </div>
</template>