# testfile
testcases
// ========================
// 📁 pages/LoginPage.ts
// ========================

export class LoginPage {
  constructor(private page: any) {}

  async goto() {
    await this.page.goto('https://www.saucedemo.com/');
  }

  async login(username: string, password: string) {
    await this.page.fill('[data-test="username"]', username);
    await this.page.fill('[data-test="password"]', password);
    await this.page.click('[data-test="login-button"]');
  }
}

// ===========================
// 📁 pages/InventoryPage.ts
// ===========================

export class InventoryPage {
  constructor(private page: any) {}

  async getInventoryItems() {
    return await this.page.locator('.inventory_item').all();
  }

  async addItemToCart(itemName: string) {
    await this.page.locator(`text=${itemName}`).locator('..').locator('button').click();
  }

  async openItemDetails(itemName: string) {
    await this.page.click(`text=${itemName}`);
  }

  async sortItems(order: string) {
    await this.page.selectOption('[data-test="product_sort_container"]', order);
  }
}

// ======================
// 📁 pages/CartPage.ts
// ======================

export class CartPage {
  constructor(private page: any) {}

  async gotoCart() {
    await this.page.click('.shopping_cart_link');
  }

  async getCartItems() {
    return await this.page.locator('.cart_item').all();
  }

  async removeItem(itemName: string) {
    await this.page.locator(`text=${itemName}`).locator('..').locator('button').click();
  }

  async proceedToCheckout() {
    await this.page.click('[data-test="checkout"]');
  }
}

// =========================
// 📁 pages/CheckoutPage.ts
// =========================

export class CheckoutPage {
  constructor(private page: any) {}

  async fillCheckoutInfo(first: string, last: string, zip: string) {
    await this.page.fill('[data-test="firstName"]', first);
    await this.page.fill('[data-test="lastName"]', last);
    await this.page.fill('[data-test="postalCode"]', zip);
    await this.page.click('[data-test="continue"]');
  }

  async finishOrder() {
    await this.page.click('[data-test="finish"]');
  }

  async getConfirmationMessage() {
    return await this.page.locator('.complete-header').textContent();
  }
}

// ==========================
// 📁 tests/ui.spec.ts
// ==========================

import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { InventoryPage } from '../pages/InventoryPage';
import { CartPage } from '../pages/CartPage';
import { CheckoutPage } from '../pages/CheckoutPage';

test.describe('UI Test Suite for SauceDemo', () => {
  let loginPage: LoginPage;
  let inventoryPage: InventoryPage;
  let cartPage: CartPage;
  let checkoutPage: CheckoutPage;

  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    inventoryPage = new InventoryPage(page);
    cartPage = new CartPage(page);
    checkoutPage = new CheckoutPage(page);
    await loginPage.goto();
  });

  test('TC01: Valid login', async ({ page }) => {
    await loginPage.login('standard_user', 'secret_sauce');
    await expect(page).toHaveURL(/inventory/);
  });

  test('TC02: Invalid login', async ({ page }) => {
    await loginPage.login('invalid_user', 'wrong_password');
    await expect(page.locator('[data-test="error"]')).toBeVisible();
  });

  test('TC03: Inventory items are visible', async ({ page }) => {
    await loginPage.login('standard_user', 'secret_sauce');
    const items = await inventoryPage.getInventoryItems();
    expect(items.length).toBeGreaterThan(0);
  });

  test('TC04: Add product to cart', async ({ page }) => {
    await loginPage.login('standard_user', 'secret_sauce');
    await inventoryPage.addItemToCart('Sauce Labs Backpack');
    await cartPage.gotoCart();
    const cartItems = await cartPage.getCartItems();
    expect(cartItems.length).toBe(1);
  });

  test('TC05: Remove product from cart', async ({ page }) => {
    await loginPage.login('standard_user', 'secret_sauce');
    await inventoryPage.addItemToCart('Sauce Labs Bike Light');
    await cartPage.gotoCart();
    await cartPage.removeItem('Sauce Labs Bike Light');
    const cartItems = await cartPage.getCartItems();
    expect(cartItems.length).toBe(0);
  });

  test('TC06: Cart badge reflects item count', async ({ page }) => {
    await loginPage.login('standard_user', 'secret_sauce');
    await inventoryPage.addItemToCart('Sauce Labs Bolt T-Shirt');
    await expect(page.locator('.shopping_cart_badge')).toHaveText('1');
  });

  test('TC07: Proceed to checkout', async ({ page }) => {
    await loginPage.login('standard_user', 'secret_sauce');
    await inventoryPage.addItemToCart('Sauce Labs Onesie');
    await cartPage.gotoCart();
    await cartPage.proceedToCheckout();
    await expect(page).toHaveURL(/checkout-step-one/);
  });

  test('TC08: Complete purchase', async ({ page }) => {
    await loginPage.login('standard_user', 'secret_sauce');
    await inventoryPage.addItemToCart('Sauce Labs Fleece Jacket');
    await cartPage.gotoCart();
    await cartPage.proceedToCheckout();
    await checkoutPage.fillCheckoutInfo('Priti', 'Roti', '12345');
    await checkoutPage.finishOrder();
    const message = await checkoutPage.getConfirmationMessage();
    expect(message).toContain('Thank you for your order');
  });

  test('TC09: Validate sorting by Price (low to high)', async ({ page }) => {
    await loginPage.login('standard_user', 'secret_sauce');
    await inventoryPage.sortItems('lohi');
    const prices = await page.$$eval('.inventory_item_price', items => items.map(i => parseFloat(i.textContent.replace('$',''))));
    const sorted = [...prices].sort((a, b) => a - b);
    expect(prices).toEqual(sorted);
  });

  test('TC10: Logout redirects to login page', async ({ page }) => {
    await loginPage.login('standard_user', 'secret_sauce');
    await page.click('#react-burger-menu-btn');
    await page.click('#logout_sidebar_link');
    await expect(page).toHaveURL('https://www.saucedemo.com/');
  });
});

