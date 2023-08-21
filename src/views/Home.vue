<script setup lang="ts">
import { ref } from "vue";
import { supabase } from "@/supabase";
import User from "@/components/User.vue";

interface Order {
  name: string;
  id: string;
  created_at: string;
  client_id: string;
  price: number;
  address: string;
  city: string;
  zip: string;
}

const orders = ref<Order[]>([]);

const orders2 = ref<Order[]>([]);

const pagination = ref(20);

const insertOrder = async () => {
  try {
    const { data, error } = await supabase.from("orders").insert({
      address: "123 Main St",
      zip: "10001",
      city: "New York",
      name: "Emily Williams",
      client_id: "afe8c2a3-3d38-4c0b-afca-23ce643d50f9",
      price: 12,
    });

    if (data) {
      console.log(data);
    }
  } catch (error) {
    console.log(error);
  }
};

const fetchOrdersFromEmilyWilliams = async () => {
  try {
    const { data: orders, error } = await supabase
      .from("orders")
      .select("*")
      .eq("name", "Emily Williams");

    if (orders) {
      console.log(orders);
    }
  } catch (error) {
    console.log(error);
  }
};
//fetchOrdersFromEmilyWilliams();

const updateEmilyWilliamOrder = async () => {
  try {
    const { data, error } = await supabase
      .from("orders")
      .update({ price: "200" })
      .eq("id", "a6b7126a-92ce-4b87-aafe-37df930e5c32");

    if (data) {
      console.log(data);
    }
  } catch (error) {
    console.log(error);
  }
};

const deleteEmilyWilliamOrder = async () => {
  try {
    const { data, error } = await supabase
      .from("orders")
      .delete()
      .eq("id", "c6869cf7-557b-4390-8f84-3ffb997df98a");

    if (data) {
      console.log(data);
    }
  } catch (error) {
    console.log(error);
  }
};

//subsciptions the lastest messages coming from the database
const channel = supabase
  .channel("my_new_channel_for_order")
  .on(
    "postgres_changes",
    {
      event: "*",
      schema: "public",
      table: "orders",
    },
    (event) => {
      const { new: newOrder } = event;
      orders.value = orders.value.map((order) => {
        if (order.id === newOrder.id) {
          return { ...order, ...newOrder };
        }
        return order;
      });
    }
  )
  .subscribe();

const fetchOrders = async () => {
  try {
    const { data, error } = await supabase.from("orders2").select(
      `*,
      clients2(* 
        )
      `
    );

    if (data) {
      console.log(data);
    }
  } catch (error) {
    console.log(error);
  }
};

const incrementViews = async () => {
  try {
    const { data, error } = await supabase.rpc("increment", {
      row_id: "1f8d7ef3-be05-47cc-b933-b2594357d66b",
    });

    if (data) {
      console.log("success");
      console.log(data);
    }
  } catch (error) {
    console.log(error);
  }
};

fetchOrders();
</script>

<template>
  <main class="flex items-center justify-between">
    <div class="container px-2 mx-auto my-8">
      <User />
      <div v-for="(order, index) in orders" :key="index">
        <div
          v-if="order"
          class="grid grid-cols-5 gap-3 px-3 py-2 my-2 border rounded-lg shadow-md"
        >
          <div v-if="order.name">{{ order.name }}</div>

          <div>${{ order.price || 0 }}</div>

          <div v-if="order.address">{{ order.address }}</div>

          <div v-if="order.zip_code">{{ order.zip_code }}</div>

          <div v-if="order.city">{{ order.city }}</div>
        </div>
      </div>
    </div>
  </main>
</template>

<!-- Comment 
<div class="px-2 my-8">
      <button class="block btn btn-primary" @click="insertOrder">
        insert Orders
      </button>
      <button class="block btn btn-primary" @click="updateEmilyWilliamOrder">
        update EmilyWilliamOrder
      </button>
      <button class="block btn btn-primary" @click="deleteEmilyWilliamOrder">
        delete EmilyWilliamOrder
      </button>
    </div>
-->
