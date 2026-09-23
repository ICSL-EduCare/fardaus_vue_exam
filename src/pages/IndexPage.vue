<template>
  <q-page class="rp-page">
    <div ref="appEl" class="rp-app">
      <!-- =========================================================
           STEP 1: AUTHENTICATION (Login + Register card, same view)
           ========================================================= -->
      <section v-if="!store.currentUser" class="auth">
        <div class="auth-side gt-sm">
          <div class="brand-row">
            <q-icon name="restaurant" size="26px" />
            <span>DineDesk POS</span>
          </div>

          <h1 class="auth-headline">
            Take the order, print the bill, keep the tables moving.
          </h1>
          <p class="auth-copy">
            Menu items, table orders and printable invoices for your restaurant,
            saved right in this browser.
          </p>

          <div class="mini-ticket" aria-hidden="true">
            <div class="mt-head">Table 4, seat 2</div>
            <div class="mt-row">
              <span>2 x Beef Tehari</span><span>440.00</span>
            </div>
            <div class="mt-row">
              <span>1 x Mango Lassi</span><span>120.00</span>
            </div>
            <div class="mt-row">
              <span>3 x Butter Naan</span><span>135.00</span>
            </div>
            <div class="mt-total"><span>Total</span><span>৳695.00</span></div>
          </div>
        </div>

        <div class="auth-main">
          <div class="auth-top">
            <div class="brand-row brand-row--ink lt-md">
              <q-icon name="restaurant" size="22px" />
              <span>DineDesk POS</span>
            </div>
            <q-space />
            <q-btn
              flat
              round
              :icon="isDark ? 'light_mode' : 'dark_mode'"
              :aria-label="
                isDark ? 'Switch to light mode' : 'Switch to dark mode'
              "
              @click="store.toggleTheme()"
            >
              <q-tooltip>{{ isDark ? "Light mode" : "Dark mode" }}</q-tooltip>
            </q-btn>
          </div>

          <transition name="card-swap" mode="out-in">
            <!-- LOGIN CARD -->
            <q-card
              v-if="authMode === 'login'"
              key="login"
              flat
              class="auth-card"
            >
              <q-card-section>
                <h2 class="card-title">Log in</h2>
                <p class="card-sub">
                  Use the email and password you registered with.
                </p>
              </q-card-section>

              <q-card-section class="q-pt-none">
                <q-banner
                  v-if="loginInfo"
                  dense
                  rounded
                  class="rp-banner rp-banner--ok q-mb-md"
                >
                  <template #avatar><q-icon name="check_circle" /></template>
                  {{ loginInfo }}
                </q-banner>
                <q-banner
                  v-if="loginError"
                  dense
                  rounded
                  class="rp-banner rp-banner--error q-mb-md"
                >
                  <template #avatar><q-icon name="error_outline" /></template>
                  {{ loginError }}
                </q-banner>

                <q-form class="form-stack" @submit="submitLogin">
                  <q-input
                    v-model="loginForm.email"
                    type="email"
                    label="Email"
                    outlined
                    lazy-rules
                    autocomplete="email"
                    :rules="[rules.required, rules.email]"
                  >
                    <template #prepend><q-icon name="mail_outline" /></template>
                  </q-input>

                  <q-input
                    v-model="loginForm.password"
                    :type="showLoginPw ? 'text' : 'password'"
                    label="Password"
                    outlined
                    lazy-rules
                    autocomplete="current-password"
                    :rules="[rules.required]"
                  >
                    <template #prepend><q-icon name="lock_outline" /></template>
                    <template #append>
                      <q-icon
                        :name="showLoginPw ? 'visibility_off' : 'visibility'"
                        class="cursor-pointer"
                        @click="showLoginPw = !showLoginPw"
                      />
                    </template>
                  </q-input>

                  <q-btn
                    type="submit"
                    label="Log in"
                    color="primary"
                    unelevated
                    no-caps
                    class="full-width btn-tall"
                  />
                </q-form>
              </q-card-section>

              <q-separator />

              <q-card-section class="switch-row">
                <span>New here?</span>
                <q-btn
                  flat
                  dense
                  no-caps
                  color="primary"
                  label="Register"
                  @click="switchAuth('register')"
                />
              </q-card-section>
            </q-card>

            <!-- REGISTRATION CARD (same view, no route change) -->
            <q-card v-else key="register" flat class="auth-card">
              <q-card-section>
                <h2 class="card-title">Create your account</h2>
                <p class="card-sub">
                  Register once, then log in to set up your restaurant.
                </p>
              </q-card-section>

              <q-card-section class="q-pt-none">
                <q-banner
                  v-if="regError"
                  dense
                  rounded
                  class="rp-banner rp-banner--error q-mb-md"
                >
                  <template #avatar><q-icon name="error_outline" /></template>
                  {{ regError }}
                </q-banner>

                <q-form class="form-stack" @submit="submitRegister">
                  <q-input
                    v-model="regForm.name"
                    label="Full name"
                    outlined
                    lazy-rules
                    autocomplete="name"
                    :rules="[rules.required]"
                  >
                    <template #prepend
                      ><q-icon name="person_outline"
                    /></template>
                  </q-input>

                  <q-input
                    v-model="regForm.email"
                    type="email"
                    label="Email"
                    outlined
                    lazy-rules
                    autocomplete="email"
                    :rules="[rules.required, rules.email]"
                  >
                    <template #prepend><q-icon name="mail_outline" /></template>
                  </q-input>

                  <q-input
                    v-model="regForm.phone"
                    type="tel"
                    label="Phone"
                    outlined
                    lazy-rules
                    autocomplete="tel"
                    :rules="[rules.required, rules.phone]"
                  >
                    <template #prepend><q-icon name="call" /></template>
                  </q-input>

                  <div class="two-col">
                    <q-input
                      v-model="regForm.password"
                      :type="showRegPw ? 'text' : 'password'"
                      label="Password"
                      outlined
                      lazy-rules
                      autocomplete="new-password"
                      :rules="[rules.required, rules.min6]"
                    >
                      <template #append>
                        <q-icon
                          :name="showRegPw ? 'visibility_off' : 'visibility'"
                          class="cursor-pointer"
                          @click="showRegPw = !showRegPw"
                        />
                      </template>
                    </q-input>

                    <q-input
                      v-model="regForm.confirm"
                      :type="showRegPw ? 'text' : 'password'"
                      label="Confirm password"
                      outlined
                      lazy-rules
                      autocomplete="new-password"
                      :rules="[
                        rules.required,
                        (v) =>
                          v === regForm.password || 'Passwords do not match',
                      ]"
                    />
                  </div>

                  <q-btn
                    type="submit"
                    label="Create account"
                    color="primary"
                    unelevated
                    no-caps
                    class="full-width btn-tall"
                  />
                </q-form>
              </q-card-section>

              <q-separator />

              <q-card-section class="switch-row">
                <span>Already registered?</span>
                <q-btn
                  flat
                  dense
                  no-caps
                  color="primary"
                  label="Log in"
                  @click="switchAuth('login')"
                />
              </q-card-section>
            </q-card>
          </transition>
        </div>
      </section>

      <!-- =========================================================
           APP SHELL (after login)
           ========================================================= -->
      <section v-else class="shell">
        <header class="topbar">
          <div class="topbar-inner">
            <button type="button" class="brand-inline" @click="go('setup')">
              <span class="bi-logo">
                <img
                  v-if="restaurant && restaurant.logo"
                  :src="restaurant.logo"
                  alt=""
                />
                <q-icon v-else name="storefront" size="20px" />
              </span>
              <span class="bi-text">
                <span class="bi-name">{{
                  restaurant ? restaurant.name : "Set up your restaurant"
                }}</span>
                <span class="bi-user"
                  >Signed in as {{ store.currentUser.name }}</span
                >
              </span>
            </button>

            <q-tabs
              v-model="view"
              no-caps
              dense
              inline-label
              active-color="primary"
              indicator-color="accent"
              class="top-tabs gt-sm"
            >
              <q-tab
                v-for="t in navTabs"
                :key="t.name"
                :name="t.name"
                :icon="t.icon"
                :label="t.label"
              />
            </q-tabs>

            <q-space />

            <q-btn
              flat
              round
              :icon="isDark ? 'light_mode' : 'dark_mode'"
              :aria-label="
                isDark ? 'Switch to light mode' : 'Switch to dark mode'
              "
              @click="store.toggleTheme()"
            >
              <q-tooltip>{{ isDark ? "Light mode" : "Dark mode" }}</q-tooltip>
            </q-btn>
            <q-btn
              flat
              round
              icon="logout"
              aria-label="Log out"
              @click="logout"
            >
              <q-tooltip>Log out</q-tooltip>
            </q-btn>
          </div>
        </header>

        <main class="content">
          <!-- ===================== STEP 2: MY RESTAURANT ===================== -->
          <div v-if="view === 'setup'" class="view">
            <div class="view-head">
              <div>
                <h2 class="view-title">My restaurant</h2>
                <p class="view-sub">
                  These details print at the top of every invoice.
                </p>
              </div>
            </div>

            <div v-if="!restaurant" class="empty-block">
              <q-icon name="storefront" size="44px" />
              <div class="empty-title">No restaurant info yet</div>
              <p>
                Add your restaurant name, logo, address and branches. You can
                edit them any time.
              </p>
              <q-btn
                unelevated
                no-caps
                color="primary"
                icon="add"
                label="Add restaurant info"
                @click="openRestaurantForm"
              />
            </div>

            <div v-else class="resto">
              <div class="resto-logo">
                <img
                  v-if="restaurant.logo"
                  :src="restaurant.logo"
                  alt="Restaurant logo"
                />
                <span v-else>{{ initials(restaurant.name) }}</span>
              </div>

              <div class="resto-body">
                <div class="resto-name">{{ restaurant.name }}</div>
                <div class="resto-meta">
                  <q-icon name="place" /> {{ restaurant.address }}
                </div>
                <div class="resto-meta">
                  <q-icon name="call" /> {{ restaurant.phone }}
                </div>
                <div class="resto-meta">
                  <q-icon name="receipt" /> VAT {{ restaurant.vat || 0 }}%
                </div>
                <div class="resto-branches">
                  <span class="branch-label">Branches</span>
                  <template
                    v-if="restaurant.branches && restaurant.branches.length"
                  >
                    <q-chip
                      v-for="b in restaurant.branches"
                      :key="b"
                      dense
                      square
                      class="branch-chip"
                      >{{ b }}</q-chip
                    >
                  </template>
                  <span v-else class="muted">No branches added</span>
                </div>
              </div>

              <div class="resto-actions">
                <q-btn
                  outline
                  no-caps
                  color="primary"
                  icon="edit"
                  label="Edit"
                  @click="openRestaurantForm"
                />
                <q-btn
                  flat
                  no-caps
                  color="negative"
                  icon="delete_outline"
                  label="Delete"
                  @click="confirmDeleteRestaurant"
                />
              </div>
            </div>

            <div class="stats">
              <div class="stat">
                <span class="stat-num">{{ stats.items }}</span>
                <span class="stat-label">Menu items</span>
              </div>
              <div class="stat">
                <span class="stat-num">{{ stats.open }}</span>
                <span class="stat-label">Upcoming orders</span>
              </div>
              <div class="stat">
                <span class="stat-num">{{ money(stats.todaySales) }}</span>
                <span class="stat-label"
                  >Sales today, {{ stats.todayCount }}
                  {{ stats.todayCount === 1 ? "order" : "orders" }}</span
                >
              </div>
            </div>

            <div class="go-grid">
              <button
                v-for="t in navTabs.slice(1)"
                :key="t.name"
                type="button"
                class="go-card"
                @click="go(t.name)"
              >
                <q-icon :name="t.icon" size="26px" class="go-icon" />
                <span class="go-title">{{ t.label }}</span>
                <span class="go-text">{{ t.hint }}</span>
              </button>
            </div>
          </div>

          <!-- ===================== STEP 3: ALL ITEMS ===================== -->
          <div v-else-if="view === 'items'" class="view">
            <div class="view-head">
              <div>
                <h2 class="view-title">All items</h2>
                <p class="view-sub">
                  {{ store.myItems.length }}
                  {{ store.myItems.length === 1 ? "item" : "items" }} on your
                  menu
                </p>
              </div>
              <q-btn
                unelevated
                no-caps
                color="primary"
                icon="add"
                label="Add item"
                @click="openItemForm()"
              />
            </div>

            <div v-if="!store.myItems.length" class="empty-block">
              <q-icon name="restaurant_menu" size="44px" />
              <div class="empty-title">Your menu is empty</div>
              <p>
                Add items one by one, or load a sample menu to try the order
                flow quickly.
              </p>
              <div class="row q-gutter-sm justify-center">
                <q-btn
                  unelevated
                  no-caps
                  color="primary"
                  icon="add"
                  label="Add item"
                  @click="openItemForm()"
                />
                <q-btn
                  outline
                  no-caps
                  color="primary"
                  label="Load sample menu"
                  @click="loadSample"
                />
              </div>
            </div>

            <template v-else>
              <div class="toolbar">
                <q-input
                  v-model="itemSearch"
                  dense
                  outlined
                  clearable
                  placeholder="Search items"
                  class="tb-search"
                >
                  <template #prepend><q-icon name="search" /></template>
                </q-input>
                <q-select
                  v-model="itemCategory"
                  :options="categoryOptions"
                  dense
                  outlined
                  label="Category"
                  class="tb-select"
                />
                <q-select
                  v-model="itemSort"
                  :options="sortOptions"
                  dense
                  outlined
                  emit-value
                  map-options
                  label="Sort"
                  class="tb-select"
                />
              </div>

              <div v-if="!filteredItems.length" class="empty-inline">
                No items match your search.
                <q-btn
                  flat
                  dense
                  no-caps
                  color="primary"
                  label="Clear filters"
                  @click="clearItemFilters"
                />
              </div>

              <div class="item-grid">
                <q-card
                  v-for="item in filteredItems"
                  :key="item.id"
                  flat
                  class="item-card"
                  :style="{ '--cat': catColor(item.category) }"
                >
                  <div class="item-top">
                    <span class="item-cat">{{ item.category }}</span>
                    <div class="item-name">{{ item.name }}</div>
                  </div>
                  <div class="item-bottom">
                    <div class="item-price">{{ money(item.price) }}</div>
                    <div class="item-actions">
                      <q-btn
                        flat
                        round
                        dense
                        icon="edit"
                        aria-label="Edit item"
                        @click="openItemForm(item)"
                      >
                        <q-tooltip>Edit item</q-tooltip>
                      </q-btn>
                      <q-btn
                        flat
                        round
                        dense
                        color="negative"
                        icon="delete_outline"
                        aria-label="Delete item"
                        @click="confirmDeleteItem(item)"
                      >
                        <q-tooltip>Delete item</q-tooltip>
                      </q-btn>
                    </div>
                  </div>
                </q-card>
              </div>
            </template>
          </div>

          <!-- ===================== STEP 4: ORDER ===================== -->
          <div v-else-if="view === 'order'" class="view view--wide">
            <div class="order-layout">
              <div class="menu-pane">
                <div class="view-head">
                  <div>
                    <h2 class="view-title">New order</h2>
                    <p class="view-sub">
                      Tap an item to add it to the ticket. Tap again for more.
                    </p>
                  </div>
                </div>

                <q-banner
                  v-if="!restaurant"
                  dense
                  rounded
                  class="rp-banner rp-banner--warn q-mb-md"
                >
                  Add your restaurant info so its name and address appear on
                  invoices.
                  <template #action>
                    <q-btn
                      flat
                      dense
                      no-caps
                      label="Add info"
                      @click="go('setup')"
                    />
                  </template>
                </q-banner>

                <div v-if="!store.myItems.length" class="empty-block">
                  <q-icon name="restaurant_menu" size="44px" />
                  <div class="empty-title">No items to order yet</div>
                  <p>
                    Add items on the All items page, then come back to take
                    orders.
                  </p>
                  <q-btn
                    unelevated
                    no-caps
                    color="primary"
                    label="Go to All items"
                    @click="go('items')"
                  />
                </div>

                <template v-else>
                  <div class="toolbar">
                    <q-input
                      v-model="menuSearch"
                      dense
                      outlined
                      clearable
                      placeholder="Search menu"
                      class="tb-search"
                    >
                      <template #prepend><q-icon name="search" /></template>
                    </q-input>
                  </div>

                  <div class="chip-row">
                    <q-chip
                      v-for="c in categoryOptions"
                      :key="c"
                      clickable
                      :outline="menuCategory !== c"
                      :color="menuCategory === c ? 'primary' : undefined"
                      :text-color="menuCategory === c ? 'white' : undefined"
                      @click="menuCategory = c"
                    >
                      {{ c }}
                    </q-chip>
                  </div>

                  <div v-if="!menuItems.length" class="empty-inline">
                    No items match your search.
                  </div>

                  <div class="menu-grid">
                    <button
                      v-for="item in menuItems"
                      :key="item.id"
                      type="button"
                      class="menu-tile"
                      :class="{ 'menu-tile--picked': qtyOf(item.id) > 0 }"
                      :style="{ '--cat': catColor(item.category) }"
                      @click="addToCart(item)"
                    >
                      <span class="mt-cat">{{ item.category }}</span>
                      <span class="mt-name">{{ item.name }}</span>
                      <span class="mt-price">{{ money(item.price) }}</span>
                      <span v-if="qtyOf(item.id)" class="mt-qty">{{
                        qtyOf(item.id)
                      }}</span>
                    </button>
                  </div>
                </template>
              </div>

              <aside ref="ticketEl" class="ticket-pane">
                <div class="ticket-paper">
                  <div class="tp-head">
                    <div class="tp-title">Order ticket</div>
                    <div class="tp-sub">
                      {{ restaurant ? restaurant.name : "Your restaurant" }}
                    </div>
                  </div>

                  <q-form
                    ref="orderFormRef"
                    class="tp-form"
                    @submit="submitOrder"
                  >
                    <q-input
                      v-model="orderForm.customerName"
                      :dark="false"
                      dense
                      outlined
                      label="Customer name"
                      lazy-rules
                      :rules="[rules.required]"
                    />
                    <q-input
                      v-model="orderForm.phone"
                      :dark="false"
                      dense
                      outlined
                      type="tel"
                      label="Phone number"
                      lazy-rules
                      :rules="[rules.required, rules.phone]"
                    />
                    <div class="two-col two-col--always">
                      <q-input
                        v-model="orderForm.table"
                        :dark="false"
                        dense
                        outlined
                        type="number"
                        min="1"
                        label="Table no."
                        lazy-rules
                        :rules="[rules.required, rules.positive]"
                      />
                      <q-input
                        v-model="orderForm.seat"
                        :dark="false"
                        dense
                        outlined
                        type="number"
                        min="1"
                        label="Seat no."
                        lazy-rules
                        :rules="[rules.required, rules.positive]"
                      />
                    </div>
                    <q-select
                      v-if="
                        restaurant &&
                        restaurant.branches &&
                        restaurant.branches.length
                      "
                      v-model="orderForm.branch"
                      :options="restaurant.branches"
                      :dark="false"
                      dense
                      outlined
                      clearable
                      label="Branch (optional)"
                      class="q-mb-md"
                    />

                    <div class="tp-rule"></div>

                    <div v-if="!cart.length" class="tp-empty">
                      No items yet. Tap a menu item to add it here.
                    </div>

                    <div
                      v-for="line in cart"
                      :key="line.itemId"
                      class="tp-line"
                    >
                      <div class="tpl-name">
                        {{ line.name }}
                        <small>{{ money(line.price) }} each</small>
                      </div>
                      <div class="tpl-qty">
                        <q-btn
                          flat
                          round
                          dense
                          size="sm"
                          icon="remove"
                          aria-label="Decrease quantity"
                          @click="changeQty(line, -1)"
                        />
                        <span>{{ line.qty }}</span>
                        <q-btn
                          flat
                          round
                          dense
                          size="sm"
                          icon="add"
                          aria-label="Increase quantity"
                          @click="changeQty(line, 1)"
                        />
                      </div>
                      <div class="tpl-total">
                        {{ money(line.price * line.qty) }}
                      </div>
                    </div>

                    <div class="tp-rule"></div>

                    <div class="tp-sum">
                      <span>Subtotal</span
                      ><span>{{ money(cartSubtotal) }}</span>
                    </div>
                    <div v-if="vatRate" class="tp-sum">
                      <span>VAT ({{ vatRate }}%)</span
                      ><span>{{ money(cartVat) }}</span>
                    </div>
                    <div class="tp-total">
                      <span>Total</span><span>{{ money(cartTotal) }}</span>
                    </div>

                    <div class="tp-actions">
                      <q-btn
                        flat
                        no-caps
                        label="Clear"
                        :disable="!cart.length"
                        @click="clearCart"
                      />
                      <q-btn
                        type="submit"
                        unelevated
                        no-caps
                        color="primary"
                        icon="check"
                        label="Place order"
                        class="col"
                        :disable="!cart.length"
                      />
                    </div>
                  </q-form>
                </div>
              </aside>
            </div>

            <q-btn
              v-if="cart.length"
              unelevated
              no-caps
              rounded
              color="accent"
              text-color="dark"
              icon="receipt_long"
              class="ticket-fab lt-md"
              :label="`Ticket (${cartCount}) ${money(cartTotal)}`"
              @click="scrollToTicket"
            />
          </div>

          <!-- ===================== STEP 5: INVOICES ===================== -->
          <div v-else class="view">
            <div class="view-head">
              <div>
                <h2 class="view-title">Invoices</h2>
                <p class="view-sub">
                  Upcoming orders are today's orders not yet served. Everything
                  older, or served, moves to previous orders.
                </p>
              </div>
              <q-btn
                unelevated
                no-caps
                color="primary"
                icon="add"
                label="New order"
                @click="go('order')"
              />
            </div>

            <div v-if="!store.myOrders.length" class="empty-block">
              <q-icon name="receipt_long" size="44px" />
              <div class="empty-title">No orders yet</div>
              <p>Place an order and its invoice shows up here.</p>
              <q-btn
                unelevated
                no-caps
                color="primary"
                label="Take an order"
                @click="go('order')"
              />
            </div>

            <template v-else>
              <div class="toolbar">
                <q-input
                  v-model="invSearch"
                  dense
                  outlined
                  clearable
                  placeholder="Search customer, phone or invoice no."
                  class="tb-search"
                >
                  <template #prepend><q-icon name="search" /></template>
                </q-input>
                <q-select
                  v-model="invRange"
                  :options="rangeOptions"
                  dense
                  outlined
                  emit-value
                  map-options
                  label="Date"
                  class="tb-select"
                />
              </div>

              <div
                v-for="sec in invoiceSections"
                :key="sec.key"
                class="inv-section"
              >
                <div class="sec-head">
                  <h3 class="sec-title">{{ sec.title }}</h3>
                  <span class="sec-count">{{ sec.orders.length }}</span>
                  <span class="sec-note">{{ sec.note }}</span>
                </div>

                <div v-if="!sec.orders.length" class="empty-inline">
                  {{ sec.empty }}
                </div>

                <div class="order-list">
                  <div
                    v-for="o in sec.orders"
                    :key="o.id"
                    class="order-row"
                    :class="{ 'order-row--fresh': o.id === lastPlacedId }"
                  >
                    <div class="or-id">
                      <div class="or-no">{{ o.invoiceNo }}</div>
                      <div class="or-time">{{ fmtDateTime(o.createdAt) }}</div>
                    </div>
                    <div class="or-cust">
                      <div class="or-name">{{ o.customerName }}</div>
                      <div class="or-meta">
                        {{ o.phone }}, table {{ o.table }}, seat {{ o.seat
                        }}<span v-if="o.branch">, {{ o.branch }}</span>
                      </div>
                      <div class="or-meta">
                        {{ itemCount(o) }}
                        {{ itemCount(o) === 1 ? "item" : "items" }}
                        <span v-if="o.status === 'served'" class="or-tag"
                          >Served</span
                        >
                      </div>
                    </div>
                    <div class="or-total">{{ money(o.total) }}</div>
                    <div class="or-actions">
                      <q-btn
                        flat
                        dense
                        no-caps
                        icon="visibility"
                        :label="$q.screen.gt.xs ? 'View' : undefined"
                        aria-label="View invoice"
                        @click="openInvoice(o)"
                      />
                      <q-btn
                        flat
                        dense
                        no-caps
                        icon="download"
                        :label="$q.screen.gt.xs ? 'Download' : undefined"
                        aria-label="Download invoice"
                        @click="downloadInvoice(o)"
                      />
                      <q-btn
                        flat
                        dense
                        no-caps
                        icon="print"
                        :label="$q.screen.gt.xs ? 'Print' : undefined"
                        aria-label="Print invoice"
                        @click="printInvoice(o)"
                      />
                      <q-btn
                        flat
                        round
                        dense
                        icon="more_vert"
                        aria-label="More actions"
                      >
                        <q-menu auto-close>
                          <q-list dense style="min-width: 190px">
                            <q-item
                              v-if="isUpcoming(o)"
                              clickable
                              @click="markServed(o)"
                            >
                              <q-item-section avatar
                                ><q-icon name="done_all"
                              /></q-item-section>
                              <q-item-section>Mark as served</q-item-section>
                            </q-item>
                            <q-item
                              v-else-if="
                                o.status === 'served' && isToday(o.createdAt)
                              "
                              clickable
                              @click="reopenOrder(o)"
                            >
                              <q-item-section avatar
                                ><q-icon name="undo"
                              /></q-item-section>
                              <q-item-section>Move to upcoming</q-item-section>
                            </q-item>
                            <q-item
                              clickable
                              class="text-negative"
                              @click="confirmDeleteOrder(o)"
                            >
                              <q-item-section avatar
                                ><q-icon name="delete_outline"
                              /></q-item-section>
                              <q-item-section>Delete order</q-item-section>
                            </q-item>
                          </q-list>
                        </q-menu>
                      </q-btn>
                    </div>
                  </div>
                </div>
              </div>
            </template>
          </div>
        </main>

        <nav class="bottom-nav lt-md">
          <q-tabs
            v-model="view"
            no-caps
            dense
            active-color="primary"
            indicator-color="transparent"
            class="bottom-tabs"
          >
            <q-tab
              v-for="t in navTabs"
              :key="t.name"
              :name="t.name"
              :icon="t.icon"
              :label="t.short"
            />
          </q-tabs>
        </nav>
      </section>
    </div>

    <!-- ===================== DIALOG: RESTAURANT FORM ===================== -->
    <q-dialog v-model="restoDialog" persistent>
      <q-card class="dlg-card">
        <div class="dlg-head">
          <div class="dlg-title">
            {{ restaurant ? "Edit restaurant info" : "Add restaurant info" }}
          </div>
          <q-btn
            v-close-popup
            flat
            round
            dense
            icon="close"
            aria-label="Close"
          />
        </div>

        <q-form @submit="saveRestaurant">
          <q-card-section class="form-stack dlg-body">
            <div class="logo-picker">
              <div class="lp-preview">
                <img
                  v-if="restoForm.logo"
                  :src="restoForm.logo"
                  alt="Logo preview"
                />
                <q-icon v-else name="image" size="28px" />
              </div>
              <div class="lp-controls">
                <q-file
                  v-model="logoFile"
                  accept="image/*"
                  dense
                  outlined
                  label="Upload logo"
                  @update:model-value="onLogoPicked"
                >
                  <template #prepend><q-icon name="upload" /></template>
                </q-file>
                <div class="row items-center q-gutter-sm">
                  <span class="hint"
                    >PNG or JPG. It is resized to fit browser storage.</span
                  >
                  <q-btn
                    v-if="restoForm.logo"
                    flat
                    dense
                    no-caps
                    color="negative"
                    label="Remove logo"
                    @click="clearLogo"
                  />
                </div>
              </div>
            </div>

            <q-input
              v-model="restoForm.name"
              outlined
              label="Restaurant name"
              lazy-rules
              :rules="[rules.required]"
            />
            <q-input
              v-model="restoForm.address"
              outlined
              autogrow
              type="textarea"
              label="Address"
              lazy-rules
              :rules="[rules.required]"
            />
            <div class="two-col">
              <q-input
                v-model="restoForm.phone"
                outlined
                type="tel"
                label="Phone number"
                lazy-rules
                :rules="[rules.required, rules.phone]"
              />
              <q-input
                v-model="restoForm.vat"
                outlined
                type="number"
                min="0"
                max="100"
                step="0.5"
                label="VAT %"
                hint="Use 0 for no VAT"
                lazy-rules
                :rules="[rules.vat]"
              />
            </div>

            <div>
              <q-input
                v-model="branchInput"
                outlined
                dense
                label="Branch name"
                hint="Press Enter or Add. You can add several branches."
                @keydown.enter.prevent="addBranch"
              >
                <template #append>
                  <q-btn
                    flat
                    dense
                    no-caps
                    color="primary"
                    label="Add"
                    @click="addBranch"
                  />
                </template>
              </q-input>
              <div v-if="restoForm.branches.length" class="q-mt-md">
                <q-chip
                  v-for="(b, i) in restoForm.branches"
                  :key="b"
                  removable
                  square
                  class="branch-chip"
                  @remove="restoForm.branches.splice(i, 1)"
                >
                  {{ b }}
                </q-chip>
              </div>
            </div>
          </q-card-section>

          <q-card-actions align="right" class="dlg-actions">
            <q-btn v-close-popup flat no-caps label="Cancel" />
            <q-btn
              type="submit"
              unelevated
              no-caps
              color="primary"
              :label="restaurant ? 'Save changes' : 'Add restaurant'"
            />
          </q-card-actions>
        </q-form>
      </q-card>
    </q-dialog>

    <!-- ===================== DIALOG: ITEM FORM ===================== -->
    <q-dialog v-model="itemDialog">
      <q-card class="dlg-card">
        <div class="dlg-head">
          <div class="dlg-title">
            {{ itemForm.id ? "Edit item" : "Add item" }}
          </div>
          <q-btn
            v-close-popup
            flat
            round
            dense
            icon="close"
            aria-label="Close"
          />
        </div>

        <q-form @submit="saveItem">
          <q-card-section class="form-stack dlg-body">
            <q-input
              v-model="itemForm.name"
              outlined
              autofocus
              label="Item name"
              lazy-rules
              :rules="[rules.required]"
            />
            <q-select
              v-model="itemForm.category"
              outlined
              label="Category"
              use-input
              hide-selected
              fill-input
              input-debounce="0"
              :options="catOptions"
              hint="Pick one or type a new category"
              lazy-rules
              :rules="[rules.required]"
              @filter="filterCats"
              @input-value="(v) => (itemForm.category = v)"
            />
            <q-input
              v-model="itemForm.price"
              outlined
              type="number"
              min="0"
              step="0.01"
              prefix="৳"
              label="Price"
              lazy-rules
              :rules="[rules.price]"
            />
          </q-card-section>

          <q-card-actions align="right" class="dlg-actions">
            <q-btn v-close-popup flat no-caps label="Cancel" />
            <q-btn
              type="submit"
              unelevated
              no-caps
              color="primary"
              :label="itemForm.id ? 'Save changes' : 'Add item'"
            />
          </q-card-actions>
        </q-form>
      </q-card>
    </q-dialog>

    <!-- ===================== DIALOG: VIEW INVOICE ===================== -->
    <q-dialog v-model="invoiceOpen" :maximized="$q.screen.lt.sm">
      <div v-if="invoiceOrder" class="inv-dialog">
        <div class="receipt">
          <div class="rc-head">
            <img
              v-if="invRestaurant.logo"
              :src="invRestaurant.logo"
              alt=""
              class="rc-logo"
            />
            <div class="rc-name">{{ invRestaurant.name }}</div>
            <div v-if="invRestaurant.address" class="rc-small">
              {{ invRestaurant.address }}
            </div>
            <div v-if="invRestaurant.phone" class="rc-small">
              Phone: {{ invRestaurant.phone }}
            </div>
            <div v-if="invoiceOrder.branch" class="rc-small">
              Branch: {{ invoiceOrder.branch }}
            </div>
          </div>

          <div class="rc-rule"></div>

          <div class="rc-kv">
            <span>Invoice</span><b>{{ invoiceOrder.invoiceNo }}</b>
          </div>
          <div class="rc-kv">
            <span>Date</span
            ><span>{{ fmtDateTime(invoiceOrder.createdAt) }}</span>
          </div>
          <div class="rc-kv">
            <span>Customer</span><span>{{ invoiceOrder.customerName }}</span>
          </div>
          <div class="rc-kv">
            <span>Phone</span><span>{{ invoiceOrder.phone }}</span>
          </div>
          <div class="rc-kv">
            <span>Table / seat</span
            ><span>{{ invoiceOrder.table }} / {{ invoiceOrder.seat }}</span>
          </div>

          <div class="rc-rule"></div>

          <table class="rc-table">
            <thead>
              <tr>
                <th>Item</th>
                <th class="c">Qty</th>
                <th class="r">Price</th>
                <th class="r">Total</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="l in invoiceOrder.lines" :key="l.itemId">
                <td>{{ l.name }}</td>
                <td class="c">{{ l.qty }}</td>
                <td class="r">{{ plain(l.price) }}</td>
                <td class="r">{{ plain(l.price * l.qty) }}</td>
              </tr>
            </tbody>
          </table>

          <div class="rc-rule"></div>

          <div class="rc-kv">
            <span>Subtotal</span><span>{{ money(invoiceOrder.subtotal) }}</span>
          </div>
          <div v-if="invoiceOrder.vatRate" class="rc-kv">
            <span>VAT ({{ invoiceOrder.vatRate }}%)</span
            ><span>{{ money(invoiceOrder.vat) }}</span>
          </div>
          <div class="rc-kv rc-grand">
            <span>Total</span><span>{{ money(invoiceOrder.total) }}</span>
          </div>

          <div class="rc-foot">Thank you for dining with us.</div>
        </div>

        <div class="inv-actions">
          <q-btn v-close-popup flat no-caps label="Close" />
          <q-space />
          <q-btn
            outline
            no-caps
            color="primary"
            icon="download"
            label="Download"
            @click="downloadInvoice(invoiceOrder)"
          />
          <q-btn
            unelevated
            no-caps
            color="primary"
            icon="print"
            label="Print"
            @click="printInvoice(invoiceOrder)"
          />
        </div>
      </div>
    </q-dialog>

    <!-- ===================== DIALOG: CONFIRM ===================== -->
    <q-dialog v-model="confirmBox.open">
      <q-card class="dlg-card dlg-card--sm">
        <q-card-section>
          <div class="dlg-title">{{ confirmBox.title }}</div>
          <p class="dlg-text">{{ confirmBox.message }}</p>
        </q-card-section>
        <q-card-actions align="right" class="dlg-actions">
          <q-btn v-close-popup flat no-caps label="Cancel" />
          <q-btn
            unelevated
            no-caps
            :color="confirmBox.danger ? 'negative' : 'primary'"
            :label="confirmBox.ok"
            @click="runConfirm"
          />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <!-- ===================== TOAST ===================== -->
    <transition name="toast">
      <div
        v-if="toast.show"
        class="rp-toast"
        :class="`rp-toast--${toast.type}`"
        role="status"
      >
        <q-icon
          :name="toast.type === 'error' ? 'error_outline' : 'check_circle'"
          size="20px"
        />
        <span>{{ toast.text }}</span>
      </div>
    </transition>
  </q-page>
</template>

<script>
/* =============================================================
   PINIA STORE + LOCAL STORAGE
   (kept in this SFC so the whole app lives in one file)
   ============================================================= */
import { defineStore } from "pinia";

const LS = {
  users: "rpos_users",
  session: "rpos_session",
  restaurants: "rpos_restaurants",
  items: "rpos_items",
  orders: "rpos_orders",
  theme: "rpos_theme",
};

function readLS(key, fallback) {
  try {
    const raw = localStorage.getItem(key);
    return raw === null ? fallback : JSON.parse(raw);
  } catch {
    return fallback;
  }
}

function writeLS(key, value) {
  try {
    localStorage.setItem(key, JSON.stringify(value));
    return true;
  } catch (err) {
    console.error("Local Storage write failed:", err);
    return false;
  }
}

const uid = () =>
  typeof crypto !== "undefined" && crypto.randomUUID
    ? crypto.randomUUID()
    : Date.now().toString(36) + Math.random().toString(36).slice(2, 10);

const round2 = (n) => Math.round((Number(n) + Number.EPSILON) * 100) / 100;

// Simple one-way hash (cyrb53) so passwords are not saved as plain text.
// There is no backend here, so this is obfuscation only, not real security.
function hashPassword(str, seed = 7) {
  let h1 = 0xdeadbeef ^ seed;
  let h2 = 0x41c6ce57 ^ seed;
  for (let i = 0; i < str.length; i++) {
    const ch = str.charCodeAt(i);
    h1 = Math.imul(h1 ^ ch, 2654435761);
    h2 = Math.imul(h2 ^ ch, 1597334677);
  }
  h1 = Math.imul(h1 ^ (h1 >>> 16), 2246822507);
  h1 ^= Math.imul(h2 ^ (h2 >>> 13), 3266489909);
  h2 = Math.imul(h2 ^ (h2 >>> 16), 2246822507);
  h2 ^= Math.imul(h1 ^ (h1 >>> 13), 3266489909);
  return (4294967296 * (2097151 & h2) + (h1 >>> 0)).toString(16);
}

const SAMPLE_MENU = [
  { name: "Beef Tehari", category: "Rice", price: 220 },
  { name: "Chicken Biryani", category: "Rice", price: 250 },
  { name: "Plain Naan", category: "Bread", price: 30 },
  { name: "Butter Naan", category: "Bread", price: 45 },
  { name: "Chicken Tikka", category: "Grill", price: 280 },
  { name: "Beef Seekh Kebab", category: "Grill", price: 260 },
  { name: "Mixed Vegetable", category: "Curry", price: 150 },
  { name: "Mango Lassi", category: "Drinks", price: 120 },
  { name: "Borhani", category: "Drinks", price: 80 },
  { name: "Firni", category: "Dessert", price: 90 },
];

export const usePosStore = defineStore("restaurant-pos", {
  state: () => ({
    users: readLS(LS.users, []),
    sessionEmail: readLS(LS.session, null),
    restaurants: readLS(LS.restaurants, {}),
    items: readLS(LS.items, []),
    orders: readLS(LS.orders, []),
    theme: readLS(LS.theme, "light"),
  }),

  getters: {
    currentUser: (s) => s.users.find((u) => u.email === s.sessionEmail) || null,
    restaurant: (s) =>
      (s.sessionEmail && s.restaurants[s.sessionEmail]) || null,
    myItems: (s) => s.items.filter((i) => i.owner === s.sessionEmail),
    myOrders: (s) => s.orders.filter((o) => o.owner === s.sessionEmail),
    categories() {
      return [
        ...new Set(this.myItems.map((i) => i.category).filter(Boolean)),
      ].sort((a, b) => a.localeCompare(b));
    },
  },

  actions: {
    /* ---------- auth ---------- */
    register({ name, email, phone, password }) {
      const e = email.trim().toLowerCase();
      if (this.users.some((u) => u.email === e)) {
        throw new Error(
          "An account with this email already exists. Log in instead.",
        );
      }
      this.users.push({
        id: uid(),
        name: name.trim(),
        email: e,
        phone: phone.trim(),
        password: hashPassword(e + ":" + password),
        createdAt: new Date().toISOString(),
      });
      writeLS(LS.users, this.users);
      return e;
    },

    login(email, password) {
      const e = email.trim().toLowerCase();
      const user = this.users.find((u) => u.email === e);
      if (!user)
        throw new Error("No account found for this email. Register first.");
      if (user.password !== hashPassword(e + ":" + password)) {
        throw new Error("Incorrect password. Check it and try again.");
      }
      this.sessionEmail = e;
      writeLS(LS.session, e);
    },

    logout() {
      this.sessionEmail = null;
      localStorage.removeItem(LS.session);
    },

    /* ---------- restaurant ---------- */
    saveRestaurant(info) {
      const next = {
        ...this.restaurants,
        [this.sessionEmail]: { ...info, updatedAt: new Date().toISOString() },
      };
      if (!writeLS(LS.restaurants, next)) {
        throw new Error(
          "Could not save. Browser storage is full; try a smaller logo.",
        );
      }
      this.restaurants = next;
    },

    deleteRestaurant() {
      const next = { ...this.restaurants };
      delete next[this.sessionEmail];
      this.restaurants = next;
      writeLS(LS.restaurants, next);
    },

    /* ---------- items ---------- */
    saveItem({ id, name, category, price }) {
      const clash = this.myItems.find(
        (i) => i.name.toLowerCase() === name.toLowerCase() && i.id !== id,
      );
      if (clash) throw new Error(`"${name}" is already on your menu.`);

      const now = new Date().toISOString();
      if (id) {
        const item = this.items.find((i) => i.id === id);
        if (item)
          Object.assign(item, { name, category, price, updatedAt: now });
      } else {
        this.items.push({
          id: uid(),
          owner: this.sessionEmail,
          name,
          category,
          price,
          createdAt: now,
        });
      }
      writeLS(LS.items, this.items);
    },

    deleteItem(id) {
      this.items = this.items.filter((i) => i.id !== id);
      writeLS(LS.items, this.items);
    },

    loadSampleMenu() {
      const existing = new Set(this.myItems.map((i) => i.name.toLowerCase()));
      const now = Date.now();
      SAMPLE_MENU.forEach((s, idx) => {
        if (!existing.has(s.name.toLowerCase())) {
          this.items.push({
            id: uid(),
            owner: this.sessionEmail,
            ...s,
            createdAt: new Date(now + idx).toISOString(),
          });
        }
      });
      writeLS(LS.items, this.items);
    },

    /* ---------- orders ---------- */
    placeOrder({ customerName, phone, table, seat, branch, lines }) {
      const r = this.restaurant;
      const seq =
        this.myOrders.reduce((m, o) => Math.max(m, o.seq || 0), 0) + 1;
      const subtotal = round2(lines.reduce((s, l) => s + l.price * l.qty, 0));
      const vatRate = Number(r && r.vat) || 0;
      const vat = round2((subtotal * vatRate) / 100);

      this.orders.push({
        id: uid(),
        owner: this.sessionEmail,
        seq,
        invoiceNo: "INV-" + String(seq).padStart(4, "0"),
        customerName: String(customerName).trim(),
        phone: String(phone).trim(),
        table: String(table),
        seat: String(seat),
        branch: branch || "",
        lines: lines.map((l) => ({
          itemId: l.itemId,
          name: l.name,
          category: l.category,
          price: l.price,
          qty: l.qty,
        })),
        subtotal,
        vatRate,
        vat,
        total: round2(subtotal + vat),
        status: "upcoming",
        createdAt: new Date().toISOString(),
        // snapshot so old invoices keep the details they were printed with
        restaurant: {
          name: (r && r.name) || "",
          address: (r && r.address) || "",
          phone: (r && r.phone) || "",
        },
      });
      writeLS(LS.orders, this.orders);
      return this.orders[this.orders.length - 1];
    },

    setOrderStatus(id, status) {
      const o = this.orders.find((x) => x.id === id);
      if (o) {
        o.status = status;
        writeLS(LS.orders, this.orders);
      }
    },

    deleteOrder(id) {
      this.orders = this.orders.filter((o) => o.id !== id);
      writeLS(LS.orders, this.orders);
    },

    /* ---------- theme ---------- */
    toggleTheme() {
      this.theme = this.theme === "dark" ? "light" : "dark";
      writeLS(LS.theme, this.theme);
    },
  },
});
</script>

<script setup>
import {
  ref,
  reactive,
  computed,
  watch,
  nextTick,
  onMounted,
  onBeforeUnmount,
} from "vue";
import { useQuasar, setCssVar } from "quasar";
import { createPinia, setActivePinia, getActivePinia } from "pinia";

// Uses the app's Pinia if Quasar created one; otherwise starts one here.
let pinia = getActivePinia();
if (!pinia) {
  pinia = createPinia();
  setActivePinia(pinia);
}
const store = usePosStore(pinia);
const $q = useQuasar();

/* ---------------- theme ---------------- */
const isDark = computed(() => store.theme === "dark");

function applyTheme(dark) {
  $q.dark.set(dark);
  setCssVar("primary", dark ? "#3F8F77" : "#1F5E4E");
  setCssVar("accent", "#E0A030");
  setCssVar("negative", dark ? "#E0604C" : "#B83E2C");
  setCssVar("positive", "#2E8B57");
}
watch(isDark, applyTheme, { immediate: true });

onMounted(() => {
  if (!document.getElementById("rp-fonts")) {
    const link = document.createElement("link");
    link.id = "rp-fonts";
    link.rel = "stylesheet";
    link.href =
      "https://fonts.googleapis.com/css2?family=Bricolage+Grotesque:opsz,wght@12..96,600;12..96,800&family=Figtree:wght@400;500;600;700&family=IBM+Plex+Mono:wght@400;600&display=swap";
    document.head.appendChild(link);
  }
});

/* ---------------- helpers ---------------- */
const money = (v) =>
  "৳" +
  Number(v || 0).toLocaleString("en-US", {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
  });
const plain = (v) =>
  Number(v || 0).toLocaleString("en-US", {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
  });
const fmtDateTime = (iso) =>
  new Date(iso).toLocaleString("en-GB", {
    day: "2-digit",
    month: "short",
    year: "numeric",
    hour: "2-digit",
    minute: "2-digit",
  });
const isToday = (iso) =>
  new Date(iso).toDateString() === new Date().toDateString();
const initials = (name = "") =>
  name
    .split(/\s+/)
    .filter(Boolean)
    .slice(0, 2)
    .map((w) => w[0].toUpperCase())
    .join("");
const itemCount = (o) => o.lines.reduce((s, l) => s + l.qty, 0);
const escapeHtml = (s) =>
  String(s ?? "").replace(
    /[&<>"']/g,
    (c) =>
      ({ "&": "&amp;", "<": "&lt;", ">": "&gt;", '"': "&quot;", "'": "&#39;" })[
        c
      ],
  );

const CAT_COLORS = [
  "#2F7D6B",
  "#C9822B",
  "#7A5AA6",
  "#B5473A",
  "#3C6FA8",
  "#6B8E23",
  "#A0527A",
  "#4E7C8A",
];
function catColor(c = "") {
  let h = 0;
  for (const ch of c) h = (h * 31 + ch.charCodeAt(0)) >>> 0;
  return CAT_COLORS[h % CAT_COLORS.length];
}

const rules = {
  required: (v) =>
    (v !== null && v !== undefined && String(v).trim() !== "") ||
    "This field is required",
  email: (v) =>
    /^[^\s@]+@[^\s@]+\.[^\s@]{2,}$/.test(String(v).trim()) ||
    "Enter a valid email address",
  phone: (v) =>
    /^\+?[0-9][0-9\s-]{6,17}$/.test(String(v).trim()) ||
    "Enter a valid phone number",
  min6: (v) => String(v).length >= 6 || "Use at least 6 characters",
  positive: (v) =>
    (Number.isInteger(Number(v)) && Number(v) > 0) ||
    "Enter a whole number above 0",
  price: (v) =>
    (v !== "" && v !== null && !isNaN(Number(v)) && Number(v) > 0) ||
    "Enter a price above 0",
  vat: (v) =>
    v === "" ||
    v === null ||
    (Number(v) >= 0 && Number(v) <= 100) ||
    "Enter a value from 0 to 100",
};

/* ---------------- toast + confirm ---------------- */
const toast = reactive({ show: false, text: "", type: "ok" });
let toastTimer = null;
function notify(text, type = "ok") {
  toast.text = text;
  toast.type = type;
  toast.show = true;
  clearTimeout(toastTimer);
  toastTimer = setTimeout(() => (toast.show = false), 2800);
}

const confirmBox = reactive({
  open: false,
  title: "",
  message: "",
  ok: "Delete",
  danger: true,
  action: null,
});
function ask(opts) {
  Object.assign(
    confirmBox,
    { ok: "Delete", danger: true, action: null },
    opts,
    { open: true },
  );
}
function runConfirm() {
  const fn = confirmBox.action;
  confirmBox.open = false;
  if (fn) fn();
}

/* ---------------- navigation ---------------- */
const navTabs = [
  {
    name: "setup",
    label: "My restaurant",
    short: "Restaurant",
    icon: "storefront",
    hint: "",
  },
  {
    name: "items",
    label: "All items",
    short: "Items",
    icon: "restaurant_menu",
    hint: "Add, edit and remove menu items.",
  },
  {
    name: "order",
    label: "New order",
    short: "Order",
    icon: "add_shopping_cart",
    hint: "Pick items and take a table order.",
  },
  {
    name: "invoices",
    label: "Invoices",
    short: "Invoices",
    icon: "receipt_long",
    hint: "View, download and print bills.",
  },
];
const view = ref("setup");
const appEl = ref(null);

function go(name) {
  view.value = name;
}
watch(view, () =>
  nextTick(() => {
    if (appEl.value) appEl.value.scrollTop = 0;
  }),
);

const restaurant = computed(() => store.restaurant);

/* =============================================================
   STEP 1: AUTH
   ============================================================= */
const authMode = ref("login");
const loginForm = reactive({ email: "", password: "" });
const regForm = reactive({
  name: "",
  email: "",
  phone: "",
  password: "",
  confirm: "",
});
const loginError = ref("");
const loginInfo = ref("");
const regError = ref("");
const showLoginPw = ref(false);
const showRegPw = ref(false);

function switchAuth(mode) {
  authMode.value = mode;
  loginError.value = "";
  regError.value = "";
  if (mode === "register") loginInfo.value = "";
}

function submitLogin() {
  loginError.value = "";
  try {
    store.login(loginForm.email, loginForm.password);
    loginForm.password = "";
    loginInfo.value = "";
    view.value = "setup";
    notify(`Welcome, ${store.currentUser.name.split(" ")[0]}`);
  } catch (e) {
    loginError.value = e.message;
  }
}

function submitRegister() {
  regError.value = "";
  try {
    const email = store.register(regForm);
    Object.assign(regForm, {
      name: "",
      email: "",
      phone: "",
      password: "",
      confirm: "",
    });
    loginForm.email = email;
    loginForm.password = "";
    loginInfo.value = "Account created. Log in with your email and password.";
    authMode.value = "login";
  } catch (e) {
    regError.value = e.message;
  }
}

function logout() {
  store.logout();
  cart.value = [];
  view.value = "setup";
  authMode.value = "login";
  notify("Logged out");
}

/* =============================================================
   STEP 2: RESTAURANT SETUP
   ============================================================= */
const restoDialog = ref(false);
const restoForm = reactive({
  name: "",
  logo: "",
  address: "",
  phone: "",
  branches: [],
  vat: 5,
});
const logoFile = ref(null);
const branchInput = ref("");

const stats = computed(() => {
  const todays = store.myOrders.filter((o) => isToday(o.createdAt));
  return {
    items: store.myItems.length,
    open: store.myOrders.filter(isUpcoming).length,
    todaySales: todays.reduce((s, o) => s + o.total, 0),
    todayCount: todays.length,
  };
});

function openRestaurantForm() {
  const r = restaurant.value;
  Object.assign(
    restoForm,
    r
      ? {
          name: r.name,
          logo: r.logo || "",
          address: r.address,
          phone: r.phone,
          branches: [...(r.branches || [])],
          vat: r.vat ?? 0,
        }
      : { name: "", logo: "", address: "", phone: "", branches: [], vat: 5 },
  );
  logoFile.value = null;
  branchInput.value = "";
  restoDialog.value = true;
}

function onLogoPicked(file) {
  if (!file) return;
  if (!file.type.startsWith("image/")) {
    notify("Choose an image file (PNG or JPG).", "error");
    logoFile.value = null;
    return;
  }
  const reader = new FileReader();
  reader.onload = () => {
    const img = new Image();
    img.onload = () => {
      const max = 240;
      const scale = Math.min(
        1,
        max / Math.max(img.width || max, img.height || max),
      );
      const canvas = document.createElement("canvas");
      canvas.width = Math.max(1, Math.round((img.width || max) * scale));
      canvas.height = Math.max(1, Math.round((img.height || max) * scale));
      canvas.getContext("2d").drawImage(img, 0, 0, canvas.width, canvas.height);
      restoForm.logo = canvas.toDataURL("image/png");
    };
    img.onerror = () =>
      notify("That file could not be read as an image.", "error");
    img.src = reader.result;
  };
  reader.readAsDataURL(file);
}

function clearLogo() {
  restoForm.logo = "";
  logoFile.value = null;
}

function addBranch() {
  const b = branchInput.value.trim();
  if (!b) return;
  if (restoForm.branches.some((x) => x.toLowerCase() === b.toLowerCase())) {
    notify(`"${b}" is already in the list.`, "error");
    return;
  }
  restoForm.branches.push(b);
  branchInput.value = "";
}

function saveRestaurant() {
  if (branchInput.value.trim()) addBranch();
  const isNew = !restaurant.value;
  try {
    store.saveRestaurant({
      name: restoForm.name.trim(),
      logo: restoForm.logo,
      address: restoForm.address.trim(),
      phone: String(restoForm.phone).trim(),
      branches: [...restoForm.branches],
      vat: Number(restoForm.vat) || 0,
    });
    restoDialog.value = false;
    notify(isNew ? "Restaurant info added" : "Restaurant info updated");
  } catch (e) {
    notify(e.message, "error");
  }
}

function confirmDeleteRestaurant() {
  ask({
    title: "Delete restaurant info?",
    message:
      "The name, logo, address and branches will be removed. Your items and invoices stay.",
    ok: "Delete info",
    action: () => {
      store.deleteRestaurant();
      notify("Restaurant info deleted");
    },
  });
}

/* =============================================================
   STEP 3: ALL ITEMS
   ============================================================= */
const itemSearch = ref("");
const itemCategory = ref("All");
const itemSort = ref("name");
const sortOptions = [
  { label: "Name (A to Z)", value: "name" },
  { label: "Price: low to high", value: "price-asc" },
  { label: "Price: high to low", value: "price-desc" },
  { label: "Newest first", value: "new" },
];
const categoryOptions = computed(() => ["All", ...store.categories]);

const filteredItems = computed(() => {
  const q = (itemSearch.value || "").trim().toLowerCase();
  const list = store.myItems.filter(
    (i) =>
      (itemCategory.value === "All" || i.category === itemCategory.value) &&
      (!q ||
        i.name.toLowerCase().includes(q) ||
        i.category.toLowerCase().includes(q)),
  );
  const sorters = {
    name: (a, b) => a.name.localeCompare(b.name),
    "price-asc": (a, b) => a.price - b.price,
    "price-desc": (a, b) => b.price - a.price,
    new: (a, b) => new Date(b.createdAt) - new Date(a.createdAt),
  };
  return [...list].sort(sorters[itemSort.value]);
});

// reset a category filter if that category no longer exists
watch(categoryOptions, (opts) => {
  if (!opts.includes(itemCategory.value)) itemCategory.value = "All";
  if (!opts.includes(menuCategory.value)) menuCategory.value = "All";
});

function clearItemFilters() {
  itemSearch.value = "";
  itemCategory.value = "All";
}

const itemDialog = ref(false);
const itemForm = reactive({ id: null, name: "", category: "", price: "" });
const catOptions = ref([]);

function openItemForm(item) {
  Object.assign(
    itemForm,
    item
      ? {
          id: item.id,
          name: item.name,
          category: item.category,
          price: item.price,
        }
      : {
          id: null,
          name: "",
          category: itemCategory.value !== "All" ? itemCategory.value : "",
          price: "",
        },
  );
  catOptions.value = [...store.categories];
  itemDialog.value = true;
}

function filterCats(val, update) {
  update(() => {
    const q = (val || "").toLowerCase();
    catOptions.value = store.categories.filter((c) =>
      c.toLowerCase().includes(q),
    );
  });
}

function saveItem() {
  const isNew = !itemForm.id;
  try {
    store.saveItem({
      id: itemForm.id,
      name: itemForm.name.trim(),
      category: String(itemForm.category || "").trim(),
      price: round2(itemForm.price),
    });
    itemDialog.value = false;
    notify(isNew ? "Item added" : "Item updated");
  } catch (e) {
    notify(e.message, "error");
  }
}

function confirmDeleteItem(item) {
  ask({
    title: `Delete ${item.name}?`,
    message:
      "It will be removed from your menu. Past invoices keep their copy of this item.",
    ok: "Delete item",
    action: () => {
      store.deleteItem(item.id);
      cart.value = cart.value.filter((l) => l.itemId !== item.id);
      notify("Item deleted");
    },
  });
}

function loadSample() {
  store.loadSampleMenu();
  notify("Sample menu added");
}

/* =============================================================
   STEP 4: ORDER
   ============================================================= */
const menuSearch = ref("");
const menuCategory = ref("All");
const cart = ref([]);
const orderForm = reactive({
  customerName: "",
  phone: "",
  table: "",
  seat: "",
  branch: null,
});
const orderFormRef = ref(null);
const ticketEl = ref(null);

const menuItems = computed(() => {
  const q = (menuSearch.value || "").trim().toLowerCase();
  return store.myItems
    .filter(
      (i) =>
        (menuCategory.value === "All" || i.category === menuCategory.value) &&
        (!q || i.name.toLowerCase().includes(q)),
    )
    .sort((a, b) => a.name.localeCompare(b.name));
});

const vatRate = computed(
  () => Number(restaurant.value && restaurant.value.vat) || 0,
);
const cartSubtotal = computed(() =>
  round2(cart.value.reduce((s, l) => s + l.price * l.qty, 0)),
);
const cartVat = computed(() =>
  round2((cartSubtotal.value * vatRate.value) / 100),
);
const cartTotal = computed(() => round2(cartSubtotal.value + cartVat.value));
const cartCount = computed(() => cart.value.reduce((s, l) => s + l.qty, 0));

function qtyOf(id) {
  const line = cart.value.find((l) => l.itemId === id);
  return line ? line.qty : 0;
}

function addToCart(item) {
  const line = cart.value.find((l) => l.itemId === item.id);
  if (line) line.qty++;
  else
    cart.value.push({
      itemId: item.id,
      name: item.name,
      category: item.category,
      price: item.price,
      qty: 1,
    });
}

function changeQty(line, delta) {
  line.qty += delta;
  if (line.qty <= 0) cart.value = cart.value.filter((l) => l !== line);
}

function clearCart() {
  cart.value = [];
}

function scrollToTicket() {
  if (ticketEl.value)
    ticketEl.value.scrollIntoView({ behavior: "smooth", block: "start" });
}

function submitOrder() {
  if (!cart.value.length) {
    notify("Add at least one item to the ticket.", "error");
    return;
  }
  const order = store.placeOrder({ ...orderForm, lines: cart.value });
  cart.value = [];
  Object.assign(orderForm, {
    customerName: "",
    phone: "",
    table: "",
    seat: "",
    branch: null,
  });
  if (orderFormRef.value) orderFormRef.value.resetValidation();

  notify(`Order placed. Invoice ${order.invoiceNo} is ready.`);
  flagFresh(order.id);
  go("invoices");
  openInvoice(order);
}

/* =============================================================
   STEP 5: INVOICES
   ============================================================= */
const invSearch = ref("");
const invRange = ref("all");
const rangeOptions = [
  { label: "All time", value: "all" },
  { label: "Today", value: "today" },
  { label: "Last 7 days", value: "7d" },
  { label: "Last 30 days", value: "30d" },
];
const lastPlacedId = ref(null);
let freshTimer = null;

function flagFresh(id) {
  lastPlacedId.value = id;
  clearTimeout(freshTimer);
  freshTimer = setTimeout(() => (lastPlacedId.value = null), 5000);
}

function isUpcoming(o) {
  return o.status === "upcoming" && isToday(o.createdAt);
}

function inRange(iso) {
  if (invRange.value === "all") return true;
  if (invRange.value === "today") return isToday(iso);
  const days = invRange.value === "7d" ? 7 : 30;
  return Date.now() - new Date(iso).getTime() <= days * 86400000;
}

const filteredOrders = computed(() => {
  const q = (invSearch.value || "").trim().toLowerCase();
  return store.myOrders
    .filter(
      (o) =>
        inRange(o.createdAt) &&
        (!q ||
          o.customerName.toLowerCase().includes(q) ||
          o.phone.toLowerCase().includes(q) ||
          o.invoiceNo.toLowerCase().includes(q)),
    )
    .sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt)); // latest first
});

const invoiceSections = computed(() => {
  const filtering = !!(
    (invSearch.value || "").trim() || invRange.value !== "all"
  );
  return [
    {
      key: "upcoming",
      title: "Upcoming orders",
      note: "Latest first",
      orders: filteredOrders.value.filter(isUpcoming),
      empty: filtering
        ? "No upcoming orders match your search."
        : "No upcoming orders right now.",
    },
    {
      key: "previous",
      title: "Previous orders",
      note: "Older or served orders",
      orders: filteredOrders.value.filter((o) => !isUpcoming(o)),
      empty: filtering
        ? "No previous orders match your search."
        : "No previous orders yet.",
    },
  ];
});

function markServed(o) {
  store.setOrderStatus(o.id, "served");
  notify(`${o.invoiceNo} marked as served`);
}

function reopenOrder(o) {
  store.setOrderStatus(o.id, "upcoming");
  notify(`${o.invoiceNo} moved to upcoming`);
}

function confirmDeleteOrder(o) {
  ask({
    title: `Delete ${o.invoiceNo}?`,
    message: "This order and its invoice will be removed from this browser.",
    ok: "Delete order",
    action: () => {
      store.deleteOrder(o.id);
      notify("Order deleted");
    },
  });
}

/* ---------- view / download / print ---------- */
const invoiceOpen = ref(false);
const invoiceOrder = ref(null);

function restaurantFor(o) {
  const cur = restaurant.value || {};
  const snap = o.restaurant || {};
  return {
    name: snap.name || cur.name || "Restaurant",
    address: snap.address || cur.address || "",
    phone: snap.phone || cur.phone || "",
    logo: cur.logo || "",
  };
}
const invRestaurant = computed(() =>
  invoiceOrder.value ? restaurantFor(invoiceOrder.value) : {},
);

function openInvoice(o) {
  invoiceOrder.value = o;
  invoiceOpen.value = true;
}

function invoiceText(o) {
  const W = 44;
  const rule = "-".repeat(W);
  const center = (s) => {
    s = String(s);
    return " ".repeat(Math.max(0, Math.floor((W - s.length) / 2))) + s;
  };
  const lr = (l, r) => {
    l = String(l);
    r = String(r);
    const gap = W - l.length - r.length;
    return gap >= 1
      ? l + " ".repeat(gap) + r
      : l + "\n" + " ".repeat(Math.max(0, W - r.length)) + r;
  };
  const r = restaurantFor(o);
  const out = [center(r.name)];
  if (r.address) out.push(center(r.address));
  if (r.phone) out.push(center("Phone: " + r.phone));
  if (o.branch) out.push(center("Branch: " + o.branch));
  out.push(rule);
  out.push(lr("Invoice", o.invoiceNo));
  out.push(lr("Date", fmtDateTime(o.createdAt)));
  out.push(lr("Customer", o.customerName));
  out.push(lr("Phone", o.phone));
  out.push(lr("Table / seat", `${o.table} / ${o.seat}`));
  out.push(rule);
  o.lines.forEach((l) => {
    out.push(l.name);
    out.push(lr(`  ${l.qty} x ${plain(l.price)}`, plain(l.price * l.qty)));
  });
  out.push(rule);
  out.push(lr("Subtotal", plain(o.subtotal)));
  if (o.vatRate) out.push(lr(`VAT (${o.vatRate}%)`, plain(o.vat)));
  out.push(lr("TOTAL (BDT)", plain(o.total)));
  out.push(rule);
  out.push(center("Thank you for dining with us."));
  return out.join("\n");
}

function downloadInvoice(o) {
  const blob = new Blob([invoiceText(o)], { type: "text/plain;charset=utf-8" });
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = `${o.invoiceNo}.txt`;
  document.body.appendChild(a);
  a.click();
  a.remove();
  setTimeout(() => URL.revokeObjectURL(url), 1000);
  notify(`${o.invoiceNo}.txt downloaded`);
}

function invoiceHtml(o) {
  const e = escapeHtml;
  const r = restaurantFor(o);
  const rows = o.lines
    .map(
      (l) =>
        `<tr><td>${e(l.name)}</td><td class="c">${l.qty}</td><td class="r">${plain(l.price)}</td><td class="r">${plain(
          l.price * l.qty,
        )}</td></tr>`,
    )
    .join("");
  const kv = (k, v, cls = "") =>
    `<div class="kv ${cls}"><span>${e(k)}</span><span>${e(v)}</span></div>`;

  return `<!doctype html><html><head><meta charset="utf-8"><title>${e(o.invoiceNo)}</title>
<style>
  body{font-family:'IBM Plex Mono',ui-monospace,Menlo,Consolas,monospace;color:#111;margin:0;padding:18px;font-size:12px}
  .wrap{max-width:320px;margin:0 auto}
  .logo{display:block;margin:0 auto 6px;max-width:64px;max-height:64px}
  h1{font-size:16px;margin:0 0 4px;text-align:center}
  .muted{text-align:center;margin:0;color:#333}
  hr{border:0;border-top:1px dashed #555;margin:10px 0}
  .kv{display:flex;justify-content:space-between;gap:12px;margin:3px 0}
  .kv span:last-child{text-align:right}
  table{width:100%;border-collapse:collapse}
  th{font-weight:600;text-align:left;border-bottom:1px dashed #555;padding:4px 0}
  td{padding:3px 0;vertical-align:top}
  .c{text-align:center}.r{text-align:right}
  .grand{font-size:15px;font-weight:700;margin-top:6px}
  .foot{text-align:center;margin-top:14px}
</style></head><body><div class="wrap">
  ${r.logo ? `<img class="logo" src="${r.logo}" alt="">` : ""}
  <h1>${e(r.name)}</h1>
  ${r.address ? `<p class="muted">${e(r.address)}</p>` : ""}
  ${r.phone ? `<p class="muted">Phone: ${e(r.phone)}</p>` : ""}
  ${o.branch ? `<p class="muted">Branch: ${e(o.branch)}</p>` : ""}
  <hr>
  ${kv("Invoice", o.invoiceNo)}
  ${kv("Date", fmtDateTime(o.createdAt))}
  ${kv("Customer", o.customerName)}
  ${kv("Phone", o.phone)}
  ${kv("Table / seat", `${o.table} / ${o.seat}`)}
  <hr>
  <table><thead><tr><th>Item</th><th class="c">Qty</th><th class="r">Price</th><th class="r">Total</th></tr></thead>
  <tbody>${rows}</tbody></table>
  <hr>
  ${kv("Subtotal", money(o.subtotal))}
  ${o.vatRate ? kv(`VAT (${o.vatRate}%)`, money(o.vat)) : ""}
  ${kv("Total", money(o.total), "grand")}
  <p class="foot">Thank you for dining with us.</p>
</div></body></html>`;
}

function printInvoice(o) {
  const frame = document.createElement("iframe");
  frame.setAttribute("aria-hidden", "true");
  frame.style.cssText =
    "position:fixed;right:0;bottom:0;width:0;height:0;border:0;";
  document.body.appendChild(frame);

  const doc = frame.contentWindow.document;
  doc.open();
  doc.write(invoiceHtml(o));
  doc.close();

  const run = () => {
    frame.contentWindow.focus();
    frame.contentWindow.print();
    setTimeout(() => frame.remove(), 1500);
  };
  const img = doc.querySelector("img");
  if (img && !img.complete) {
    img.onload = run;
    img.onerror = run;
  } else {
    setTimeout(run, 60);
  }
}

onBeforeUnmount(() => {
  clearTimeout(toastTimer);
  clearTimeout(freshTimer);
});
</script>

<!-- Theme tokens live on <body> so teleported dialogs get them too -->
<style>
body {
  --rp-bg: #edf1ee;
  --rp-surface: #ffffff;
  --rp-surface-2: #f5f8f6;
  --rp-ink: #17231f;
  --rp-muted: #5b6964;
  --rp-line: #d6ded9;
  --rp-pine: #1f5e4e;
  --rp-pine-deep: #174a3d;
  --rp-saffron: #e0a030;
  --rp-chili: #b83e2c;
  --rp-paper: #ffffff;
  --rp-paper-ink: #1a1a1a;
  --rp-shadow:
    0 1px 2px rgba(23, 35, 31, 0.06), 0 6px 18px rgba(23, 35, 31, 0.06);
  --rp-font:
    "Figtree", system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
  --rp-display: "Bricolage Grotesque", "Figtree", system-ui, sans-serif;
  --rp-mono: "IBM Plex Mono", ui-monospace, Menlo, Consolas, monospace;
}

body.body--dark {
  --rp-bg: #0f1513;
  --rp-surface: #171f1c;
  --rp-surface-2: #1d2723;
  --rp-ink: #e6ece9;
  --rp-muted: #97a59f;
  --rp-line: #2a3531;
  --rp-pine: #3f8f77;
  --rp-pine-deep: #123a30;
  --rp-chili: #e0604c;
  --rp-paper: #efefe8;
  --rp-paper-ink: #1a1a1a;
  --rp-shadow: 0 1px 2px rgba(0, 0, 0, 0.4), 0 8px 22px rgba(0, 0, 0, 0.3);
}
</style>

<style scoped>
/* ---------- base ---------- */
.rp-page {
  min-height: 100vh;
}

/* Covers the default Quasar layout so this page works when pasted alone */
.rp-app {
  position: fixed;
  inset: 0;
  z-index: 2100;
  overflow-y: auto;
  background: var(--rp-bg);
  color: var(--rp-ink);
  font-family: var(--rp-font);
  -webkit-font-smoothing: antialiased;
}

.muted {
  color: var(--rp-muted);
}

.form-stack {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.two-col {
  display: grid;
  grid-template-columns: 1fr;
  gap: 0 12px;
}

@media (min-width: 600px) {
  .two-col {
    grid-template-columns: 1fr 1fr;
  }
}

.two-col--always {
  grid-template-columns: 1fr 1fr;
}

.btn-tall {
  min-height: 46px;
  font-weight: 600;
  font-size: 15px;
  margin-top: 6px;
}

/* ---------- auth ---------- */
.auth {
  min-height: 100%;
  display: grid;
  grid-template-columns: 1fr;
}

@media (min-width: 1024px) {
  .auth {
    grid-template-columns: 1.05fr 1fr;
  }
}

.auth-side {
  position: relative;
  overflow: hidden;
  background: var(--rp-pine-deep);
  color: #eef5f1;
  padding: 48px 56px;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.brand-row {
  display: flex;
  align-items: center;
  gap: 10px;
  font-family: var(--rp-display);
  font-weight: 800;
  font-size: 20px;
  letter-spacing: -0.01em;
}

.brand-row--ink {
  color: var(--rp-ink);
}

.auth-headline {
  font-family: var(--rp-display);
  font-weight: 800;
  font-size: 44px;
  line-height: 1.05;
  letter-spacing: -0.03em;
  margin: 36px 0 16px;
  max-width: 14ch;
}

.auth-copy {
  font-size: 17px;
  line-height: 1.55;
  max-width: 40ch;
  color: rgba(238, 245, 241, 0.82);
  margin: 0;
}

.mini-ticket {
  position: relative;
  width: 280px;
  margin-top: 44px;
  padding: 18px 18px 22px;
  background: #ffffff;
  color: #1a1a1a;
  font-family: var(--rp-mono);
  font-size: 13px;
  transform: rotate(-3deg);
  box-shadow: 0 18px 40px rgba(0, 0, 0, 0.25);
}

.mini-ticket::after {
  content: "";
  position: absolute;
  left: 0;
  right: 0;
  bottom: -10px;
  height: 10px;
  background: conic-gradient(
      from -45deg at bottom,
      #ffffff 90deg,
      transparent 0
    )
    0 0 / 14px 100%;
}

.mt-head {
  font-weight: 600;
  padding-bottom: 8px;
  margin-bottom: 8px;
  border-bottom: 1px dashed #999;
}

.mt-row,
.mt-total {
  display: flex;
  justify-content: space-between;
  padding: 3px 0;
}

.mt-total {
  margin-top: 8px;
  padding-top: 8px;
  border-top: 1px dashed #999;
  font-weight: 600;
  font-size: 15px;
}

.auth-main {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 24px 16px 40px;
}

.auth-top {
  width: 100%;
  max-width: 460px;
  display: flex;
  align-items: center;
  margin-bottom: 16px;
}

.auth-card {
  width: 100%;
  max-width: 460px;
  background: var(--rp-surface);
  border: 1px solid var(--rp-line);
  border-radius: 14px;
  box-shadow: var(--rp-shadow);
}

.card-title {
  font-family: var(--rp-display);
  font-weight: 800;
  font-size: 28px;
  line-height: 1.15;
  letter-spacing: -0.02em;
  margin: 4px 0 6px;
}

.card-sub {
  color: var(--rp-muted);
  margin: 0;
  font-size: 15px;
}

.switch-row {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  color: var(--rp-muted);
}

.card-swap-enter-active,
.card-swap-leave-active {
  transition:
    opacity 0.18s ease,
    transform 0.18s ease;
}

.card-swap-enter-from {
  opacity: 0;
  transform: translateY(8px);
}

.card-swap-leave-to {
  opacity: 0;
  transform: translateY(-8px);
}

/* ---------- banners ---------- */
.rp-banner {
  font-size: 14px;
}

.rp-banner--ok {
  background: rgba(46, 139, 87, 0.12);
  color: var(--rp-ink);
}

.rp-banner--error {
  background: rgba(184, 62, 44, 0.12);
  color: var(--rp-ink);
}

.rp-banner--warn {
  background: rgba(224, 160, 48, 0.16);
  color: var(--rp-ink);
}

/* ---------- shell ---------- */
.topbar {
  position: sticky;
  top: 0;
  z-index: 10;
  background: var(--rp-surface);
  border-bottom: 1px solid var(--rp-line);
}

.topbar-inner {
  max-width: 1240px;
  margin: 0 auto;
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 8px 16px;
  min-height: 60px;
}

.brand-inline {
  display: flex;
  align-items: center;
  gap: 10px;
  background: none;
  border: 0;
  padding: 0;
  cursor: pointer;
  color: inherit;
  text-align: left;
  font: inherit;
  min-width: 0;
}

.bi-logo {
  width: 38px;
  height: 38px;
  flex: none;
  border-radius: 10px;
  overflow: hidden;
  display: grid;
  place-items: center;
  background: var(--rp-surface-2);
  border: 1px solid var(--rp-line);
  color: var(--rp-pine);
}

.bi-logo img {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.bi-text {
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.bi-name {
  font-family: var(--rp-display);
  font-weight: 800;
  font-size: 17px;
  line-height: 1.2;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 220px;
}

.bi-user {
  font-size: 12px;
  color: var(--rp-muted);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 220px;
}

.top-tabs {
  color: var(--rp-muted);
  margin-left: 12px;
}

.content {
  max-width: 1240px;
  margin: 0 auto;
  padding: 28px 16px 110px;
}

@media (min-width: 1024px) {
  .content {
    padding: 36px 24px 60px;
  }
}

.bottom-nav {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 2150;
  background: var(--rp-surface);
  border-top: 1px solid var(--rp-line);
}

.bottom-tabs {
  color: var(--rp-muted);
}

/* ---------- views ---------- */
.view-head {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  gap: 16px;
  flex-wrap: wrap;
  margin-bottom: 22px;
}

.view-title {
  font-family: var(--rp-display);
  font-weight: 800;
  font-size: 32px;
  line-height: 1.1;
  letter-spacing: -0.025em;
  margin: 0;
}

.view-sub {
  margin: 6px 0 0;
  color: var(--rp-muted);
  font-size: 15px;
  max-width: 62ch;
}

.toolbar {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 18px;
}

.tb-search {
  flex: 1 1 260px;
}

.tb-select {
  flex: 0 1 190px;
  min-width: 150px;
}

.empty-block {
  text-align: center;
  padding: 48px 20px;
  border: 1.5px dashed var(--rp-line);
  border-radius: 14px;
  color: var(--rp-muted);
  background: var(--rp-surface);
}

.empty-block p {
  max-width: 44ch;
  margin: 6px auto 18px;
}

.empty-title {
  font-family: var(--rp-display);
  font-weight: 800;
  font-size: 20px;
  color: var(--rp-ink);
  margin-top: 10px;
}

.empty-inline {
  color: var(--rp-muted);
  padding: 14px 0;
}

/* ---------- step 2: restaurant ---------- */
.resto {
  display: grid;
  grid-template-columns: auto 1fr;
  gap: 20px;
  padding: 22px;
  background: var(--rp-surface);
  border: 1px solid var(--rp-line);
  border-radius: 14px;
}

@media (min-width: 800px) {
  .resto {
    grid-template-columns: auto 1fr auto;
    align-items: start;
  }
}

.resto-logo {
  width: 96px;
  height: 96px;
  border-radius: 14px;
  overflow: hidden;
  background: var(--rp-surface-2);
  border: 1px solid var(--rp-line);
  display: grid;
  place-items: center;
  font-family: var(--rp-display);
  font-weight: 800;
  font-size: 32px;
  color: var(--rp-pine);
}

.resto-logo img {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.resto-name {
  font-family: var(--rp-display);
  font-weight: 800;
  font-size: 26px;
  line-height: 1.15;
  letter-spacing: -0.02em;
  margin-bottom: 8px;
}

.resto-meta {
  display: flex;
  align-items: center;
  gap: 8px;
  color: var(--rp-muted);
  margin: 4px 0;
}

.resto-branches {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 4px;
  margin-top: 12px;
}

.branch-label {
  font-weight: 600;
  margin-right: 6px;
}

.branch-chip {
  background: var(--rp-surface-2);
  border: 1px solid var(--rp-line);
  color: var(--rp-ink);
}

.resto-actions {
  grid-column: 1 / -1;
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

@media (min-width: 800px) {
  .resto-actions {
    grid-column: auto;
    flex-direction: column;
    align-items: stretch;
  }
}

.stats {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  margin: 22px 0;
  background: var(--rp-surface);
  border: 1px solid var(--rp-line);
  border-radius: 14px;
  overflow: hidden;
}

.stat {
  display: flex;
  flex-direction: column;
  gap: 2px;
  padding: 16px 20px;
  border-right: 1px solid var(--rp-line);
}

.stat:last-child {
  border-right: 0;
}

.stat-num {
  font-family: var(--rp-display);
  font-weight: 800;
  font-size: 24px;
}

.stat-label {
  color: var(--rp-muted);
  font-size: 14px;
}

.go-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 14px;
}

.go-card {
  display: grid;
  grid-template-columns: auto 1fr;
  grid-template-rows: auto auto;
  column-gap: 14px;
  align-items: center;
  text-align: left;
  padding: 20px;
  border-radius: 14px;
  border: 1px solid var(--rp-line);
  background: var(--rp-surface);
  color: var(--rp-ink);
  font: inherit;
  cursor: pointer;
  transition:
    border-color 0.15s ease,
    transform 0.15s ease;
}

.go-card:hover {
  border-color: var(--rp-pine);
}

.go-card:active {
  transform: translateY(1px);
}

.go-card:focus-visible,
.menu-tile:focus-visible,
.brand-inline:focus-visible {
  outline: 3px solid var(--rp-saffron);
  outline-offset: 2px;
}

.go-icon {
  grid-row: span 2;
  width: 48px;
  height: 48px;
  border-radius: 12px;
  background: var(--rp-surface-2);
  color: var(--rp-pine);
}

.go-title {
  font-family: var(--rp-display);
  font-weight: 800;
  font-size: 18px;
}

.go-text {
  color: var(--rp-muted);
  font-size: 14px;
}

/* ---------- step 3: items ---------- */
.item-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 14px;
}

.item-card {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  min-height: 150px;
  background: var(--rp-surface);
  border: 1px solid var(--rp-line);
  border-top: 4px solid var(--cat);
  border-radius: 12px;
}

.item-top {
  padding: 14px 16px 6px;
}

.item-cat {
  display: inline-block;
  font-size: 12px;
  font-weight: 600;
  color: var(--cat);
  margin-bottom: 4px;
}

.item-name {
  font-weight: 700;
  font-size: 17px;
  line-height: 1.3;
}

.item-bottom {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 6px 8px 10px 16px;
}

.item-price {
  font-family: var(--rp-display);
  font-weight: 800;
  font-size: 22px;
  letter-spacing: -0.01em;
}

/* ---------- step 4: order ---------- */
.order-layout {
  display: grid;
  grid-template-columns: 1fr;
  gap: 28px;
}

@media (min-width: 1024px) {
  .order-layout {
    grid-template-columns: 1fr 380px;
    align-items: start;
  }
}

.chip-row {
  display: flex;
  flex-wrap: wrap;
  gap: 2px;
  margin: -6px 0 14px -4px;
}

.menu-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
  gap: 12px;
}

.menu-tile {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 4px;
  min-height: 118px;
  padding: 14px;
  text-align: left;
  font: inherit;
  color: var(--rp-ink);
  background: var(--rp-surface);
  border: 1px solid var(--rp-line);
  border-left: 4px solid var(--cat);
  border-radius: 12px;
  cursor: pointer;
  transition:
    border-color 0.12s ease,
    background 0.12s ease;
}

.menu-tile:hover {
  background: var(--rp-surface-2);
}

.menu-tile:active {
  transform: scale(0.98);
}

.menu-tile--picked {
  border-color: var(--rp-saffron);
  border-left-color: var(--cat);
  box-shadow: inset 0 0 0 1px var(--rp-saffron);
}

.mt-cat {
  font-size: 12px;
  font-weight: 600;
  color: var(--cat);
}

.mt-name {
  font-weight: 700;
  font-size: 16px;
  line-height: 1.25;
  padding-right: 26px;
}

.mt-price {
  margin-top: auto;
  font-family: var(--rp-display);
  font-weight: 800;
  font-size: 18px;
}

.mt-qty {
  position: absolute;
  top: 10px;
  right: 10px;
  min-width: 26px;
  height: 26px;
  padding: 0 6px;
  border-radius: 13px;
  display: grid;
  place-items: center;
  background: var(--rp-saffron);
  color: #1a1a1a;
  font-weight: 700;
  font-size: 13px;
}

.ticket-pane {
  scroll-margin-top: 80px;
}

@media (min-width: 1024px) {
  .ticket-pane {
    position: sticky;
    top: 84px;
  }
}

/* the receipt: the one bold element of the app */
.ticket-paper {
  position: relative;
  background: var(--rp-paper);
  color: var(--rp-paper-ink);
  padding: 20px 20px 26px;
  border-radius: 4px 4px 0 0;
  box-shadow: var(--rp-shadow);
  margin-bottom: 12px;
}

.ticket-paper::after {
  content: "";
  position: absolute;
  left: 0;
  right: 0;
  bottom: -10px;
  height: 10px;
  background: conic-gradient(
      from -45deg at bottom,
      var(--rp-paper) 90deg,
      transparent 0
    )
    0 0 / 16px 100%;
}

.tp-head {
  text-align: center;
  padding-bottom: 14px;
  margin-bottom: 16px;
  border-bottom: 1px dashed #9aa19d;
}

.tp-title {
  font-family: var(--rp-mono);
  font-weight: 600;
  font-size: 16px;
}

.tp-sub {
  font-size: 13px;
  color: #5b6964;
}

.tp-form {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.tp-rule {
  border-top: 1px dashed #9aa19d;
  margin: 10px 0;
}

.tp-empty {
  font-family: var(--rp-mono);
  font-size: 13px;
  color: #5b6964;
  text-align: center;
  padding: 16px 0;
}

.tp-line {
  display: grid;
  grid-template-columns: 1fr auto auto;
  align-items: center;
  gap: 8px;
  padding: 6px 0;
  font-family: var(--rp-mono);
  font-size: 13px;
}

.tpl-name {
  display: flex;
  flex-direction: column;
  font-weight: 600;
  line-height: 1.3;
}

.tpl-name small {
  font-weight: 400;
  color: #5b6964;
}

.tpl-qty {
  display: flex;
  align-items: center;
  gap: 2px;
}

.tpl-qty span {
  min-width: 20px;
  text-align: center;
  font-weight: 600;
}

.tpl-total {
  min-width: 78px;
  text-align: right;
}

.tp-sum,
.tp-total {
  display: flex;
  justify-content: space-between;
  font-family: var(--rp-mono);
  font-size: 13px;
  padding: 3px 0;
}

.tp-total {
  font-size: 18px;
  font-weight: 600;
  padding-top: 8px;
}

.tp-actions {
  display: flex;
  gap: 8px;
  margin-top: 16px;
}

.ticket-fab {
  position: fixed;
  right: 16px;
  bottom: 76px;
  z-index: 2160;
  font-weight: 600;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.2);
}

/* ---------- step 5: invoices ---------- */
.inv-section {
  margin-bottom: 30px;
}

.sec-head {
  display: flex;
  align-items: baseline;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 10px;
  padding-bottom: 8px;
  border-bottom: 2px solid var(--rp-ink);
}

.sec-title {
  font-family: var(--rp-display);
  font-weight: 800;
  font-size: 22px;
  line-height: 1.2;
  letter-spacing: -0.01em;
  margin: 0;
}

.sec-count {
  font-weight: 700;
  font-size: 13px;
  min-width: 26px;
  text-align: center;
  padding: 1px 8px;
  border-radius: 10px;
  background: var(--rp-saffron);
  color: #1a1a1a;
}

.sec-note {
  color: var(--rp-muted);
  font-size: 14px;
}

.order-list {
  display: flex;
  flex-direction: column;
}

.order-row {
  display: grid;
  grid-template-columns: 1fr auto;
  grid-template-areas:
    "id total"
    "cust cust"
    "actions actions";
  gap: 6px 16px;
  padding: 14px 4px;
  border-bottom: 1px solid var(--rp-line);
  transition: background 0.6s ease;
}

@media (min-width: 900px) {
  .order-row {
    grid-template-columns: 150px 1fr auto auto;
    grid-template-areas: "id cust total actions";
    align-items: center;
  }
}

.order-row--fresh {
  background: rgba(224, 160, 48, 0.14);
}

.or-id {
  grid-area: id;
}

.or-no {
  font-family: var(--rp-mono);
  font-weight: 600;
}

.or-time {
  font-size: 13px;
  color: var(--rp-muted);
}

.or-cust {
  grid-area: cust;
  min-width: 0;
}

.or-name {
  font-weight: 700;
}

.or-meta {
  font-size: 13px;
  color: var(--rp-muted);
}

.or-tag {
  margin-left: 6px;
  padding: 0 6px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: 600;
  color: var(--rp-pine);
  border: 1px solid currentColor;
}

.or-total {
  grid-area: total;
  font-family: var(--rp-display);
  font-weight: 800;
  font-size: 19px;
  text-align: right;
}

.or-actions {
  grid-area: actions;
  display: flex;
  justify-content: flex-end;
  flex-wrap: wrap;
  gap: 2px;
}

/* ---------- dialogs ---------- */
.dlg-card {
  width: 540px;
  max-width: 94vw;
  border-radius: 14px;
  background: var(--rp-surface);
  color: var(--rp-ink);
  font-family: var(--rp-font);
}

.dlg-card--sm {
  width: 420px;
}

.dlg-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 16px 4px 20px;
}

.dlg-title {
  font-family: var(--rp-display);
  font-weight: 800;
  font-size: 22px;
  letter-spacing: -0.01em;
}

.dlg-text {
  color: var(--rp-muted);
  margin: 8px 0 0;
}

.dlg-body {
  padding: 12px 20px;
}

.dlg-actions {
  padding: 8px 16px 16px;
}

.logo-picker {
  display: flex;
  gap: 14px;
  align-items: flex-start;
  margin-bottom: 14px;
}

.lp-preview {
  width: 76px;
  height: 76px;
  flex: none;
  border-radius: 12px;
  overflow: hidden;
  border: 1.5px dashed var(--rp-line);
  display: grid;
  place-items: center;
  color: var(--rp-muted);
  background: var(--rp-surface-2);
}

.lp-preview img {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.lp-controls {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.hint {
  font-size: 12px;
  color: var(--rp-muted);
}

/* invoice viewer */
.inv-dialog {
  width: 420px;
  max-width: 100vw;
  display: flex;
  flex-direction: column;
  background: var(--rp-surface);
  color: var(--rp-ink);
  font-family: var(--rp-font);
  border-radius: 14px;
  overflow: hidden;
}

.receipt {
  flex: 1;
  overflow-y: auto;
  margin: 20px 20px 10px;
  padding: 22px 20px 26px;
  background: var(--rp-paper);
  color: var(--rp-paper-ink);
  font-family: var(--rp-mono);
  font-size: 13px;
  border: 1px solid #e3e6e4;
}

.rc-head {
  text-align: center;
}

.rc-logo {
  display: block;
  max-width: 64px;
  max-height: 64px;
  margin: 0 auto 8px;
}

.rc-name {
  font-weight: 600;
  font-size: 17px;
  margin-bottom: 2px;
}

.rc-small {
  color: #4a4f4c;
  font-size: 12px;
}

.rc-rule {
  border-top: 1px dashed #8d948f;
  margin: 12px 0;
}

.rc-kv {
  display: flex;
  justify-content: space-between;
  gap: 12px;
  padding: 2px 0;
}

.rc-kv > :last-child {
  text-align: right;
}

.rc-grand {
  font-size: 17px;
  font-weight: 600;
  padding-top: 6px;
}

.rc-table {
  width: 100%;
  border-collapse: collapse;
}

.rc-table th {
  font-weight: 600;
  text-align: left;
  padding: 4px 0;
  border-bottom: 1px dashed #8d948f;
}

.rc-table td {
  padding: 4px 0;
  vertical-align: top;
}

.rc-table .c {
  text-align: center;
}

.rc-table .r {
  text-align: right;
}

.rc-foot {
  text-align: center;
  margin-top: 16px;
}

.inv-actions {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 20px 18px;
}

/* ---------- toast ---------- */
.rp-toast {
  position: fixed;
  left: 50%;
  bottom: 88px;
  transform: translateX(-50%);
  z-index: 9000;
  display: flex;
  align-items: center;
  gap: 10px;
  max-width: calc(100vw - 32px);
  padding: 12px 18px;
  border-radius: 10px;
  font-family: var(--rp-font);
  font-weight: 600;
  font-size: 14px;
  color: #ffffff;
  background: #1f5e4e;
  box-shadow: 0 10px 28px rgba(0, 0, 0, 0.25);
}

@media (min-width: 1024px) {
  .rp-toast {
    bottom: 28px;
  }
}

.rp-toast--error {
  background: #b83e2c;
}

.toast-enter-active,
.toast-leave-active {
  transition:
    opacity 0.2s ease,
    transform 0.2s ease;
}

.toast-enter-from,
.toast-leave-to {
  opacity: 0;
  transform: translate(-50%, 10px);
}

@media (prefers-reduced-motion: reduce) {
  .card-swap-enter-active,
  .card-swap-leave-active,
  .toast-enter-active,
  .toast-leave-active,
  .go-card,
  .menu-tile,
  .order-row {
    transition: none;
  }
}

/* ---------- small screens ---------- */
@media (max-width: 599px) {
  .view-title {
    font-size: 26px;
  }

  .resto-logo {
    width: 72px;
    height: 72px;
    font-size: 24px;
  }

  .resto-name {
    font-size: 22px;
  }

  .stat {
    border-right: 0;
    border-bottom: 1px solid var(--rp-line);
  }

  .stat:last-child {
    border-bottom: 0;
  }

  .bi-name,
  .bi-user {
    max-width: 170px;
  }

  .receipt {
    margin: 12px;
  }

  .inv-dialog {
    width: 100%;
    height: 100%;
    border-radius: 0;
  }
}
</style>
