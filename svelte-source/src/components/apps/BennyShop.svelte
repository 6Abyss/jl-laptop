<script lang="ts">
  import Apps from "@shared/Apps.svelte";
  import { scale, fly, fade } from "svelte/transition";
  import { isEnvBrowser } from "@utils/misc";
  import {
    backIn,
    backOut,
    cubicIn,
    cubicOut,
    quadIn,
    quadInOut,
    quadOut,
  } from "svelte/easing";
  import { items, cart } from "@store/bennys";
  import StoreCard from "./utility/Bennys/StoreCard.svelte";
  import { flip } from "svelte/animate";
  import { fetchNui } from "@utils/eventHandler";
  import StoreCartCard from "./utility/Bennys/StoreCartCard.svelte";
  import { notifications } from "@store/notifications";
  import { onMount } from "svelte";
  let topdata = {
    title: "Bennys Shop",
    background: "#1c1b22",
    blur: true,
    blurstrength: 15,
    color: "#1b1b1b",
  };
  let categories = ["Items"];
  let currentPage = "Items";

  function handleAddCart(e) {
    if ($cart.filter((item) => item.name === e.detail.name).length === 0) {
      cart.update((c) => [...c, e.detail]);
      notifications.send("Added item to cart", "bennys");
    } else {
      cart.update((c) => {
        c.filter((item) => item.name === e.detail.name)[0].quantity += 1;
        return [...c];
      });
    }
  }

  function handleCheckout() {
    fetchNui("laptop/checkout", {
      cart: $cart,
      app: "bennys",
    }).then((res) => {
      if (res.status === "success") {
        notifications.send(res.message, "bennys", 5000);
        cart.set([]);
      } else {
        notifications.send(res.message, "bennys", 5000);
      }
    });
  }

  function handleCartChange(e: CustomEvent, name: string) {
    cart.update((c) => {
      c.filter((it) => it.name === name)[0].quantity = e.detail;
      return [...c];
    });
  }

  function handleRemoveCart(name: string) {
    cart.set($cart.filter((e) => e.name !== name));
  }

  onMount(() => {
    fetchNui("bennys/getitems", {}).then((res) => {
      items.set(res);
    });
  });
</script>

<Apps appname="bennys" {topdata} debug={false}>
  <div class="shop">
    <div class="navigation">
      <div class="left">
        {#each categories as category}
          <button
            class:active={category === currentPage}
            on:click={() => {
              currentPage = category;
            }}
          >
            {category}
          </button>
        {/each}
      </div>
      <div class="right">
        {#if $cart.length > 0 && currentPage == "Cart"}
          <div
            class="btn"
            in:fly|local={{ y: -100, easing: cubicOut, duration: 500 }}
            out:fly|local={{ y: -100, easing: cubicIn, duration: 300 }}
          >
            <button style="width: 120px;" on:click={handleCheckout}>
              <ion-icon name="bag-check" />
              Checkout</button
            >
          </div>
        {/if}

        <button
          on:click={() => {
            currentPage = "Cart";
          }}
        >
          <ion-icon name="cart" style="font-size: 20px;" />
          Cart
          {#if $cart.length > 0}
            <div
              in:scale={{ duration: 300, easing: backOut }}
              out:scale={{ duration: 300, easing: quadInOut }}
              class="circle"
            >
              {$cart.length}
            </div>
          {/if}
        </button>
      </div>
    </div>
    <div class="pages">
      {#if currentPage !== "Cart"}
        <div class="main-page">
          {#each $items as data (data.name)}
            <div class="card" class:hide={data.category !== currentPage}>
              <StoreCard
                on:addCart={handleAddCart}
                image={`nui://${data.image}`}
                name={data.name}
                title={data.label}
                stock={data.stock}
                price={data.price}
              />
            </div>
          {/each}
        </div>
      {:else}
        <div class="cart-page">
          <div class="cart">
            {#each $cart as itemcart (itemcart.name)}
              <div
                class="cart-items"
                animate:flip={{ duration: 300, easing: quadInOut }}
                in:fly|local={{ x: -300, duration: 200, easing: quadOut }}
                out:fly|local={{ x: -300, duration: 200, easing: quadIn }}
              >
                <StoreCartCard
                  on:theChange={(e) => {
                    handleCartChange(e, itemcart.name);
                  }}
                  on:remove={() => {
                    handleRemoveCart(itemcart.name);
                  }}
                  image={itemcart.image}
                  title={itemcart.label}
                  quantity={itemcart.quantity}
                  max={$items.filter((item) => item.name === itemcart.name)[0]
                    .stock}
                />
              </div>
            {:else}
              <div class="empty" transition:fade|local={{ duration: 300 }}>
                <span>Cart is empty</span>
              </div>
            {/each}
          </div>
        </div>
      {/if}
    </div>
  </div>
</Apps>

<style>
  .cart-items {
    margin-top: 5px;
    width: 100%;

    display: flex;
    justify-content: center;
  }
  .empty {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    display: flex;
    font-size: 2.3rem;
  }

  .cart-page {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  .cart {
    width: 100%;
    height: 100%;
    padding: 10px 50px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    margin-bottom: 30px;
  }
  .card.hide {
    display: none;
  }
  .main-page {
    padding: 30px;
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 0.17fr));
    row-gap: 30px;
    justify-content: flex-start;
    align-items: center;
    width: 100%;
  }
  .pages {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
  }
  .left {
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
  }
  .circle {
    padding: 2px;
    font-size: 0.9rem;
    width: 17px;
    height: 17px;
    border-radius: 50%;
    color: #1c1b22;
    background: rgb(255, 255, 255);
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .right {
    display: flex;
    justify-content: center;
  }
  .right button {
    font-weight: 600;
    color: rgb(115, 105, 172);
    gap: 5px;
    width: 100px;
    display: flex;
    align-items: center;
    justify-content: center;
    background: rgb(37, 37, 37);
  }

  button.active {
    color: rgb(115, 105, 172);
    background: rgb(37, 37, 37);
  }
  button {
    cursor: pointer;
    font-weight: 600;
    background: transparent;
    height: 45px;
    color: rgba(148, 148, 148, 0.514);
    border: none;
    padding: 10px;
    border-radius: 30px;
    margin: 10px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 1rem;
  }
  .navigation {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 30px;
  }
  .shop {
    height: 100%;
    font-family: "Bebas Neue", cursive;
    overflow-x: hidden;
  }
</style>
