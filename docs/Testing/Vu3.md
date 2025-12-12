# 🎯 Vue3前端与ASP.NET Core Web API的L2/L3测试策略

针对Vue3前端 + ASP.NET Core后端的架构，测试策略需要分层处理。让我详细说明：

## 📊 **完整的测试分层架构**

```
┌─────────────────────────────────────────────┐
│           L3: 完整端到端测试                 │
│    (Vue3前端 + 真实API + 真实UI交互)        │
├─────────────────────────────────────────────┤
│           L2: API契约测试                    │
│    (验证后端API契约，前端Mock或真实API)      │
├─────────────────────────────────────────────┤
│           L1: 前端集成测试                   │
│    (Vue组件 + Mock API)                      │
├─────────────────────────────────────────────┤
│           L0: 前端单元测试                   │
│    (Vue组件/函数独立测试)                    │
└─────────────────────────────────────────────┘
```

---

## 🧪 **L2测试：前端视角的API契约验证**

### **方案1：使用Mock Service Worker (MSW) + Jest/Vitest**
```javascript
// tests/api-contract/userApi.spec.js
import { setupServer } from 'msw/node'
import { rest } from 'msw'
import { userApi } from '@/api/userApi'

// 定义API契约
const apiContract = {
  getUser: {
    method: 'GET',
    url: '/api/users/:id',
    response: {
      status: 200,
      schema: {
        type: 'object',
        properties: {
          id: { type: 'integer' },
          name: { type: 'string' },
          email: { type: 'string', format: 'email' }
        },
        required: ['id', 'name', 'email']
      }
    }
  },
  createUser: {
    method: 'POST',
    url: '/api/users',
    response: {
      status: 201,
      schema: {
        type: 'object',
        properties: {
          id: { type: 'integer' },
          name: { type: 'string' },
          email: { type: 'string', format: 'email' },
          createdAt: { type: 'string', format: 'date-time' }
        },
        required: ['id', 'name', 'email', 'createdAt']
      }
    }
  }
}

// 创建Mock Server
const server = setupServer(
  // GET /api/users/:id
  rest.get('/api/users/:id', (req, res, ctx) => {
    const { id } = req.params
    
    // 验证请求参数
    if (!id || isNaN(parseInt(id))) {
      return res(
        ctx.status(400),
        ctx.json({
          code: 'INVALID_ID',
          message: 'Invalid user ID'
        })
      )
    }
    
    // 返回符合契约的响应
    return res(
      ctx.status(200),
      ctx.json({
        id: parseInt(id),
        name: 'John Doe',
        email: 'john@example.com',
        // 确保所有必需字段都存在
        createdAt: new Date().toISOString()
      })
    )
  }),
  
  // POST /api/users
  rest.post('/api/users', async (req, res, ctx) => {
    const body = await req.json()
    
    // 验证请求体是否符合契约
    const validation = validateRequestBody(body, apiContract.createUser)
    if (!validation.valid) {
      return res(
        ctx.status(400),
        ctx.json({
          code: 'VALIDATION_ERROR',
          message: validation.errors.join(', ')
        })
      )
    }
    
    // 返回符合契约的响应
    return res(
      ctx.status(201),
      ctx.json({
        id: 123,
        name: body.name,
        email: body.email,
        createdAt: new Date().toISOString()
      }),
      ctx.set('Location', `/api/users/123`)
    )
  })
)

describe('User API Contract Tests', () => {
  beforeAll(() => server.listen())
  afterEach(() => server.resetHandlers())
  afterAll(() => server.close())
  
  test('getUser should return data matching contract', async () => {
    // Act
    const response = await userApi.getUser(1)
    
    // Assert - 验证响应符合契约
    expect(response.status).toBe(200)
    expect(response.data).toMatchSchema(apiContract.getUser.response.schema)
    expect(response.data).toHaveProperty('id')
    expect(response.data).toHaveProperty('name')
    expect(response.data).toHaveProperty('email')
    expect(response.data.email).toMatch(/^[^@]+@[^@]+\.[^@]+$/)
  })
  
  test('createUser should validate request and return 201', async () => {
    // Arrange
    const userData = {
      name: 'Jane Doe',
      email: 'jane@example.com',
      password: 'Secure123!'
    }
    
    // Act
    const response = await userApi.createUser(userData)
    
    // Assert
    expect(response.status).toBe(201)
    expect(response.data).toMatchSchema(apiContract.createUser.response.schema)
    expect(response.headers.location).toContain('/api/users/')
  })
  
  test('createUser with invalid email should return 400', async () => {
    // Arrange
    const invalidUserData = {
      name: 'Invalid User',
      email: 'invalid-email',
      password: '123'
    }
    
    // Act
    const response = await userApi.createUser(invalidUserData)
    
    // Assert
    expect(response.status).toBe(400)
    expect(response.data).toHaveProperty('code')
    expect(response.data).toHaveProperty('message')
  })
})
```

### **方案2：使用真实后端进行契约测试**
```javascript
// tests/contract/real-api.spec.js
import axios from 'axios'
import { validateResponse } from './schemaValidator'

describe('Real API Contract Tests', () => {
  const apiClient = axios.create({
    baseURL: process.env.VITE_API_BASE_URL || 'https://api-staging.example.com',
    timeout: 10000,
    headers: {
      'Content-Type': 'application/json',
      'X-API-Key': process.env.VITE_API_KEY
    }
  })
  
  // 测试前准备
  let testUserId = null
  
  beforeAll(async () => {
    // 创建测试用户
    const response = await apiClient.post('/api/users', {
      name: 'Test User',
      email: `test_${Date.now()}@example.com`,
      password: 'Test123!'
    })
    
    testUserId = response.data.id
  })
  
  afterAll(async () => {
    // 清理测试数据
    if (testUserId) {
      await apiClient.delete(`/api/users/${testUserId}`)
    }
  })
  
  test('API health check', async () => {
    const response = await apiClient.get('/health')
    expect(response.status).toBe(200)
    expect(response.data).toHaveProperty('status', 'healthy')
  })
  
  test('GET /api/users/:id returns valid user data', async () => {
    const response = await apiClient.get(`/api/users/${testUserId}`)
    
    // 验证HTTP层面契约
    expect(response.status).toBe(200)
    expect(response.headers['content-type']).toContain('application/json')
    
    // 验证JSON Schema
    const validation = validateResponse(response.data, 'userSchema')
    expect(validation.valid).toBe(true)
    
    // 验证业务契约
    expect(response.data).toHaveProperty('id', testUserId)
    expect(response.data).toHaveProperty('name')
    expect(response.data).toHaveProperty('email')
    expect(response.data.email).toMatch(/^[^@]+@[^@]+\.[^@]+$/)
  })
  
  test('GET /api/users returns paginated list', async () => {
    const response = await apiClient.get('/api/users', {
      params: { page: 1, pageSize: 10 }
    })
    
    expect(response.status).toBe(200)
    expect(response.data).toHaveProperty('items')
    expect(response.data).toHaveProperty('total')
    expect(response.data).toHaveProperty('page')
    expect(response.data).toHaveProperty('pageSize')
    
    expect(Array.isArray(response.data.items)).toBe(true)
    
    // 验证每个item的schema
    response.data.items.forEach(item => {
      const validation = validateResponse(item, 'userSchema')
      expect(validation.valid).toBe(true)
    })
  })
})
```

---

## 🎭 **L3测试：完整的端到端测试**

### **使用Cypress进行E2E测试**
```javascript
// cypress/e2e/user-flow.cy.js
describe('User Management E2E Tests', () => {
  const API_BASE = Cypress.env('apiUrl') || 'https://api-staging.example.com'
  const testEmail = `test_${Date.now()}@example.com`
  
  beforeEach(() => {
    // 清理测试数据
    cy.request({
      method: 'DELETE',
      url: `${API_BASE}/api/test-cleanup`,
      headers: { 'X-Test-Secret': Cypress.env('testSecret') }
    })
    
    // 访问应用
    cy.visit('/')
  })
  
  it('Complete user registration and login flow', () => {
    // 1. 点击注册按钮
    cy.get('[data-test="register-button"]').click()
    
    // 2. 填写注册表单
    cy.get('[data-test="register-email"]').type(testEmail)
    cy.get('[data-test="register-name"]').type('Test User')
    cy.get('[data-test="register-password"]').type('Test123!')
    cy.get('[data-test="register-confirm-password"]').type('Test123!')
    
    // 3. 提交注册
    cy.get('[data-test="register-submit"]').click()
    
    // 4. 验证注册成功
    cy.contains('Registration successful').should('be.visible')
    cy.contains('Please check your email').should('be.visible')
    
    // 5. 模拟邮箱验证（在测试环境中）
    cy.task('getVerificationToken', testEmail).then(token => {
      cy.visit(`/verify-email?token=${token}`)
      cy.contains('Email verified successfully').should('be.visible')
    })
    
    // 6. 登录
    cy.get('[data-test="login-email"]').type(testEmail)
    cy.get('[data-test="login-password"]').type('Test123!')
    cy.get('[data-test="login-submit"]').click()
    
    // 7. 验证登录成功
    cy.contains('Welcome, Test User').should('be.visible')
    cy.get('[data-test="user-avatar"]').should('be.visible')
    
    // 8. 访问用户资料页
    cy.get('[data-test="profile-link"]').click()
    cy.url().should('include', '/profile')
    
    // 9. 验证用户信息正确显示
    cy.get('[data-test="user-name"]').should('contain', 'Test User')
    cy.get('[data-test="user-email"]').should('contain', testEmail)
    
    // 10. 更新用户信息
    cy.get('[data-test="edit-profile"]').click()
    cy.get('[data-test="edit-name"]').clear().type('Updated Name')
    cy.get('[data-test="save-profile"]').click()
    
    // 11. 验证更新成功
    cy.contains('Profile updated successfully').should('be.visible')
    cy.get('[data-test="user-name"]').should('contain', 'Updated Name')
  })
  
  it('Product search and purchase flow', () => {
    // 登录
    cy.login(testEmail, 'Test123!')
    
    // 搜索商品
    cy.get('[data-test="search-input"]').type('iPhone')
    cy.get('[data-test="search-button"]').click()
    
    // 验证搜索结果
    cy.get('[data-test="product-item"]').should('have.length.at.least', 1)
    cy.contains('iPhone').should('be.visible')
    
    // 点击第一个商品
    cy.get('[data-test="product-item"]').first().click()
    
    // 加入购物车
    cy.get('[data-test="add-to-cart"]').click()
    cy.get('[data-test="cart-count"]').should('contain', '1')
    
    // 进入购物车
    cy.get('[data-test="view-cart"]').click()
    cy.url().should('include', '/cart')
    
    // 结算
    cy.get('[data-test="checkout-button"]').click()
    cy.url().should('include', '/checkout')
    
    // 填写收货信息
    cy.get('[data-test="shipping-name"]').type('Test User')
    cy.get('[data-test="shipping-address"]').type('123 Test Street')
    cy.get('[data-test="shipping-city"]').type('Test City')
    
    // 选择支付方式
    cy.get('[data-test="payment-method-card"]').click()
    
    // 使用测试支付信息
    cy.get('[data-test="card-number"]').type('4242 4242 4242 4242')
    cy.get('[data-test="card-expiry"]').type('12/30')
    cy.get('[data-test="card-cvc"]').type('123')
    
    // 提交支付
    cy.get('[data-test="pay-button"]').click()
    
    // 验证订单成功
    cy.contains('Order confirmed').should('be.visible')
    cy.get('[data-test="order-number"]').should('be.visible')
    
    // 验证订单出现在订单列表
    cy.get('[data-test="orders-link"]').click()
    cy.get('[data-test="order-item"]').should('have.length.at.least', 1)
  })
})
```

### **Cypress配置**
```javascript
// cypress.config.js
const { defineConfig } = require('cypress')

module.exports = defineConfig({
  e2e: {
    baseUrl: 'http://localhost:5173',
    viewportWidth: 1920,
    viewportHeight: 1080,
    video: true,
    screenshotOnRunFailure: true,
    
    env: {
      apiUrl: 'https://api-staging.example.com',
      testSecret: process.env.CYPRESS_TEST_SECRET
    },
    
    setupNodeEvents(on, config) {
      // 自定义任务
      on('task', {
        getVerificationToken(email) {
          // 从测试数据库或API获取验证token
          return getTestVerificationToken(email)
        },
        
        async createTestUser(userData) {
          // 创建测试用户
          const response = await axios.post(
            `${config.env.apiUrl}/api/test-users`,
            userData
          )
          return response.data
        },
        
        async cleanupTestUser(userId) {
          // 清理测试用户
          await axios.delete(
            `${config.env.apiUrl}/api/test-users/${userId}`
          )
          return null
        }
      })
      
      // 动态配置
      config.env.apiUrl = process.env.CYPRESS_API_URL || config.env.apiUrl
      
      return config
    }
  }
})
```

---

## 🛠️ **Vue3组件的API集成测试**

### **使用Vue Test Utils + MSW**
```vue
<!-- UserProfile.vue -->
<template>
  <div>
    <div v-if="loading">Loading...</div>
    <div v-else-if="error">{{ error }}</div>
    <div v-else>
      <h2 data-test="user-name">{{ user.name }}</h2>
      <p data-test="user-email">{{ user.email }}</p>
      <button data-test="refresh-button" @click="fetchUser">Refresh</button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { getUser } from '@/api/userApi'

const loading = ref(false)
const error = ref(null)
const user = ref(null)

const fetchUser = async () => {
  loading.value = true
  error.value = null
  
  try {
    const response = await getUser(1)
    user.value = response.data
  } catch (err) {
    error.value = 'Failed to load user'
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  fetchUser()
})
</script>
```

```javascript
// tests/components/UserProfile.spec.js
import { mount } from '@vue/test-utils'
import { setupServer } from 'msw/node'
import { rest } from 'msw'
import UserProfile from '@/components/UserProfile.vue'
import { createTestingPinia } from '@pinia/testing'

// 创建Mock Server
const server = setupServer(
  rest.get('/api/users/1', (req, res, ctx) => {
    return res(
      ctx.status(200),
      ctx.json({
        id: 1,
        name: 'John Doe',
        email: 'john@example.com'
      })
    )
  }),
  
  rest.get('/api/users/999', (req, res, ctx) => {
    return res(
      ctx.status(404),
      ctx.json({
        code: 'USER_NOT_FOUND',
        message: 'User not found'
      })
    )
  })
)

describe('UserProfile.vue', () => {
  beforeAll(() => server.listen())
  afterEach(() => server.resetHandlers())
  afterAll(() => server.close())
  
  it('should display user data when API returns success', async () => {
    // Arrange
    const wrapper = mount(UserProfile, {
      global: {
        plugins: [createTestingPinia()]
      }
    })
    
    // Act - 等待API调用完成
    await wrapper.vm.fetchUser()
    await wrapper.vm.$nextTick()
    
    // Assert
    expect(wrapper.find('[data-test="user-name"]').text()).toBe('John Doe')
    expect(wrapper.find('[data-test="user-email"]').text()).toBe('john@example.com')
    expect(wrapper.find('[data-test="refresh-button"]').exists()).toBe(true)
  })
  
  it('should display error when API fails', async () => {
    // 修改handler模拟错误
    server.use(
      rest.get('/api/users/1', (req, res, ctx) => {
        return res(ctx.status(500))
      })
    )
    
    const wrapper = mount(UserProfile, {
      global: {
        plugins: [createTestingPinia()]
      }
    })
    
    await wrapper.vm.fetchUser()
    await wrapper.vm.$nextTick()
    
    expect(wrapper.text()).toContain('Failed to load user')
  })
  
  it('should reload data when refresh button clicked', async () => {
    const wrapper = mount(UserProfile, {
      global: {
        plugins: [createTestingPinia()]
      }
    })
    
    // 初始加载
    await wrapper.vm.fetchUser()
    
    // 修改API返回数据
    let callCount = 0
    server.use(
      rest.get('/api/users/1', (req, res, ctx) => {
        callCount++
        if (callCount === 1) {
          return res(
            ctx.json({ id: 1, name: 'First', email: 'first@example.com' })
          )
        } else {
          return res(
            ctx.json({ id: 1, name: 'Updated', email: 'updated@example.com' })
          )
        }
      })
    )
    
    // 点击刷新按钮
    await wrapper.find('[data-test="refresh-button"]').trigger('click')
    await wrapper.vm.$nextTick()
    
    expect(wrapper.find('[data-test="user-name"]').text()).toBe('Updated')
  })
})
```

---

## 📦 **前端测试技术栈推荐**

### **package.json配置**
```json
{
  "scripts": {
    "test:unit": "vitest",
    "test:unit:ui": "vitest --ui",
    "test:component": "vitest run tests/components/",
    "test:contract": "vitest run tests/contract/",
    "test:e2e": "cypress run",
    "test:e2e:open": "cypress open",
    "test:all": "npm run test:unit && npm run test:contract && npm run test:e2e"
  },
  "devDependencies": {
    // 单元测试
    "vitest": "^1.0.0",
    "@vue/test-utils": "^2.4.0",
    "jsdom": "^22.0.0",
    
    // Mocking
    "msw": "^2.0.0",
    
    // 断言和匹配器
    "@testing-library/jest-dom": "^6.0.0",
    "@testing-library/vue": "^7.0.0",
    "@vitest/expect": "^1.0.0",
    
    // 端到端测试
    "cypress": "^13.0.0",
    "@cypress/vite-dev-server": "^5.0.0",
    
    // 测试数据
    "@faker-js/faker": "^8.0.0",
    
    // Schema验证
    "ajv": "^8.12.0",
    "ajv-formats": "^2.1.1"
  }
}
```

### **vitest.config.js**
```javascript
import { defineConfig } from 'vitest/config'
import vue from '@vitejs/plugin-vue'
import path from 'path'

export default defineConfig({
  plugins: [vue()],
  
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./tests/setup.js'],
    coverage: {
      provider: 'istanbul',
      reporter: ['text', 'json', 'html']
    }
  },
  
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src')
    }
  }
})
```

---

## 🚀 **CI/CD流水线集成**

### **GitHub Actions完整配置**
```yaml
name: Full Test Suite
on: [push, pull_request]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run unit tests
        run: npm run test:unit
        
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        
  api-contract-tests:
    runs-on: ubuntu-latest
    needs: unit-tests
    env:
      VITE_API_BASE_URL: ${{ secrets.STAGING_API_URL }}
      VITE_API_KEY: ${{ secrets.STAGING_API_KEY }}
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        
      - name: Install dependencies
        run: npm ci
        
      - name: Run API contract tests
        run: npm run test:contract
        
      - name: Wait for API to be ready
        run: |
          for i in {1..30}; do
            if curl -s ${{ secrets.STAGING_API_URL }}/health | grep -q "healthy"; then
              echo "API is ready"
              break
            fi
            echo "Waiting for API... ($i/30)"
            sleep 2
          done
        
  e2e-tests:
    runs-on: ubuntu-latest
    needs: api-contract-tests
    env:
      CYPRESS_API_URL: ${{ secrets.STAGING_API_URL }}
      CYPRESS_TEST_SECRET: ${{ secrets.TEST_SECRET }}
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        
      - name: Install dependencies
        run: npm ci
        
      - name: Start dev server
        run: npm run dev &
        
      - name: Wait for dev server
        run: |
          for i in {1..30}; do
            if curl -s http://localhost:5173 | grep -q "DOCTYPE"; then
              echo "Dev server is ready"
              break
            fi
            echo "Waiting for dev server... ($i/30)"
            sleep 2
          done
        
      - name: Run E2E tests
        run: npm run test:e2e
        
      - name: Upload test artifacts
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: cypress-artifacts
          path: |
            cypress/screenshots
            cypress/videos
```

---

## 💡 **最佳实践总结**

### **前端测试策略**
1. **L0单元测试**：测试Vue组件逻辑、工具函数
2. **L1集成测试**：Vue组件 + Mock API (MSW)
3. **L2契约测试**：
   - **开发阶段**：使用MSW Mock API验证前端处理逻辑
   - **部署阶段**：连接真实测试环境API验证契约
4. **L3端到端测试**：Cypress测试完整用户流程

### **前后端协作测试**
```javascript
// 共享API契约定义
// shared/contracts/user-api.json
{
  "getUser": {
    "path": "/api/users/:id",
    "method": "GET",
    "response": {
      "200": {
        "schema": {
          // 与后端保持一致的JSON Schema
        }
      }
    }
  }
}

// 前端使用
import userContract from '@/contracts/user-api.json'

// 后端使用（.NET）
// 生成TypeScript类型定义供前端使用
```

### **Mock策略**
```javascript
// tests/mocks/handlers.js
export const handlers = [
  // 开发阶段：返回Mock数据
  rest.get('/api/users/:id', (req, res, ctx) => {
    return res(ctx.json(mockUser))
  }),
  
  // 契约测试阶段：验证请求/响应格式
  rest.get('/api/users/:id', async (req, res, ctx) => {
    // 验证请求参数
    validateRequest(req)
    
    // 返回符合契约的响应
    return res(ctx.json(generateValidResponse()))
  })
]
```

### **环境配置**
```env
# .env.development
VITE_API_BASE_URL=http://localhost:5000
VITE_USE_MOCK_API=true

# .env.staging
VITE_API_BASE_URL=https://api-staging.example.com
VITE_USE_MOCK_API=false

# .env.test
VITE_API_BASE_URL=https://api-staging.example.com
VITE_USE_MOCK_API=false
```

**关键点**：前端测试的重点是验证**前端代码正确处理API响应**，而不是验证后端API的实现。真正的L2契约测试应该由后端团队负责，前端可以参与定义契约，但最终验证应该在部署后的环境中进行。