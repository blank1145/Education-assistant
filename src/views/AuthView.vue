<script setup>
import { ref, computed } from "vue";
import { useRouter, useRoute } from "vue-router";
import { useEduScopeStore } from "../composables/useEduScopeStore";
import { api } from "../utils/api";

const router = useRouter();
const route = useRoute();
const store = useEduScopeStore();

const isLogin = ref(true);
const loading = ref(false);
const errorMessage = ref("");

const form = ref({
  username: "",
  email: "",
  password: "",
  confirmPassword: "",
  nickname: "",
});

const formTitle = computed(() => (isLogin.value ? "登录" : "注册"));
const switchText = computed(() =>
  isLogin.value ? "没有账号？立即注册" : "已有账号？立即登录"
);

function switchMode() {
  isLogin.value = !isLogin.value;
  errorMessage.value = "";
}

function validateForm() {
  if (isLogin.value) {
    if (!form.value.username || !form.value.password) {
      errorMessage.value = "请填写用户名和密码";
      return false;
    }
  } else {
    if (!form.value.username || form.value.username.length < 3) {
      errorMessage.value = "用户名至少需要3个字符";
      return false;
    }
    if (!/^[a-zA-Z0-9_]+$/.test(form.value.username)) {
      errorMessage.value = "用户名只能包含字母、数字和下划线";
      return false;
    }
    if (
      !form.value.email ||
      !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.value.email)
    ) {
      errorMessage.value = "请输入有效的邮箱地址";
      return false;
    }
    if (!form.value.password || form.value.password.length < 6) {
      errorMessage.value = "密码至少需要6个字符";
      return false;
    }
    if (form.value.password !== form.value.confirmPassword) {
      errorMessage.value = "两次输入的密码不一致";
      return false;
    }
  }
  return true;
}

async function handleSubmit() {
  errorMessage.value = "";

  if (!validateForm()) {
    return;
  }

  loading.value = true;

  try {
    let result;
    if (isLogin.value) {
      result = await api.login(form.value.username, form.value.password);
    } else {
      result = await api.register(
        form.value.username,
        form.value.email,
        form.value.password,
        form.value.nickname
      );
    }

    if (result.success) {
      store.setUser(result.data.user);
      store.showToast(isLogin.value ? "登录成功" : "注册成功");

      const redirect = route.query.redirect || "/";
      router.push(redirect);
    }
  } catch (error) {
    errorMessage.value = error.message || "操作失败，请稍后重试";
  } finally {
    loading.value = false;
  }
}
</script>

<template>
  <div class="auth-page">
    <div class="auth-wrapper">
      <div class="auth-brand">
        <div class="brand-icon">ea</div>
        <h1 class="brand-name">edua</h1>
        <p class="brand-desc">
          教育辅助平台 - 成绩分析、智能助手、出题与出卷工作台
        </p>
      </div>

      <div class="auth-card">
        <div class="auth-header">
          <h2 class="auth-title">{{ formTitle }}</h2>
          <p class="auth-subtitle">
            {{
              isLogin ? "欢迎回来，请登录您的账号" : "创建新账号，开始使用平台"
            }}
          </p>
        </div>

        <form class="auth-form" @submit.prevent="handleSubmit">
          <div v-if="errorMessage" class="auth-error">
            <svg
              width="16"
              height="16"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
            >
              <circle cx="12" cy="12" r="10"></circle>
              <line x1="15" y1="9" x2="9" y2="15"></line>
              <line x1="9" y1="9" x2="15" y2="15"></line>
            </svg>
            {{ errorMessage }}
          </div>

          <div class="form-group">
            <label class="form-label">
              {{ isLogin ? "用户名/邮箱" : "用户名" }}
            </label>
            <input
              v-model="form.username"
              type="text"
              class="form-input"
              :placeholder="isLogin ? '请输入用户名或邮箱' : '请输入用户名'"
              autocomplete="username"
            />
          </div>

          <div v-if="!isLogin" class="form-group">
            <label class="form-label">邮箱</label>
            <input
              v-model="form.email"
              type="email"
              class="form-input"
              placeholder="请输入邮箱"
              autocomplete="email"
            />
          </div>

          <div class="form-group">
            <label class="form-label">密码</label>
            <input
              v-model="form.password"
              type="password"
              class="form-input"
              placeholder="请输入密码"
              autocomplete="current-password"
            />
          </div>

          <div v-if="!isLogin" class="form-group">
            <label class="form-label">确认密码</label>
            <input
              v-model="form.confirmPassword"
              type="password"
              class="form-input"
              placeholder="请再次输入密码"
              autocomplete="new-password"
            />
          </div>

          <div v-if="!isLogin" class="form-group">
            <label class="form-label">昵称（可选）</label>
            <input
              v-model="form.nickname"
              type="text"
              class="form-input"
              placeholder="请输入昵称"
            />
          </div>

          <button type="submit" class="auth-button" :disabled="loading">
            <span v-if="loading" class="loading-spinner"></span>
            {{ loading ? "处理中..." : formTitle }}
          </button>

          <div class="auth-footer">
            <span class="footer-text">{{
              isLogin ? "还没有账号？" : "已有账号？"
            }}</span>
            <button type="button" class="switch-button" @click="switchMode">
              {{ isLogin ? "立即注册" : "立即登录" }}
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<style scoped>
.auth-page {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
  margin: 0;
}

.auth-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 32px;
  width: 100%;
  max-width: 480px;
}

.auth-brand {
  text-align: center;
  color: white;
}

.brand-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 64px;
  height: 64px;
  background: rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  font-size: 1.5rem;
  font-weight: 800;
  letter-spacing: 0.05em;
  margin-bottom: 16px;
}

.brand-name {
  font-size: 2rem;
  font-weight: 700;
  margin: 0 0 8px;
  letter-spacing: -0.02em;
}

.brand-desc {
  font-size: 0.95rem;
  opacity: 0.9;
  margin: 0;
  max-width: 320px;
}

.auth-card {
  width: 100%;
  background: white;
  border-radius: 24px;
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.25);
  padding: 40px 48px;
}

.auth-header {
  text-align: center;
  margin-bottom: 32px;
}

.auth-title {
  font-size: 1.75rem;
  font-weight: 700;
  color: #1a1a2e;
  margin: 0 0 8px;
}

.auth-subtitle {
  font-size: 0.95rem;
  color: #6b7280;
  margin: 0;
}

.auth-form {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.auth-error {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #dc2626;
  padding: 12px 16px;
  border-radius: 12px;
  font-size: 0.875rem;
  text-align: center;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.form-label {
  font-size: 0.875rem;
  font-weight: 600;
  color: #374151;
}

.form-input {
  padding: 14px 16px;
  border: 2px solid #e5e7eb;
  border-radius: 12px;
  font-size: 1rem;
  background: #f9fafb;
  transition: all 0.2s ease;
}

.form-input:focus {
  outline: none;
  border-color: #667eea;
  background: white;
  box-shadow: 0 0 0 4px rgba(102, 126, 234, 0.1);
}

.form-input::placeholder {
  color: #9ca3af;
}

.auth-button {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  margin-top: 8px;
  padding: 16px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 12px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
}

.auth-button:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(102, 126, 234, 0.4);
}

.auth-button:active:not(:disabled) {
  transform: translateY(0);
}

.auth-button:disabled {
  opacity: 0.7;
  cursor: not-allowed;
  transform: none;
}

.loading-spinner {
  width: 16px;
  height: 16px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-top-color: white;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.auth-footer {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  margin-top: 8px;
  padding-top: 20px;
  border-top: 1px solid #e5e7eb;
}

.footer-text {
  font-size: 0.875rem;
  color: #6b7280;
}

.switch-button {
  background: none;
  border: none;
  color: #667eea;
  font-size: 0.875rem;
  font-weight: 600;
  cursor: pointer;
  padding: 0;
  transition: color 0.2s;
}

.switch-button:hover {
  color: #764ba2;
  text-decoration: underline;
}

@media (max-width: 520px) {
  .auth-page {
    padding: 16px;
  }

  .auth-wrapper {
    gap: 24px;
  }

  .brand-icon {
    width: 56px;
    height: 56px;
    font-size: 1.25rem;
  }

  .brand-name {
    font-size: 1.75rem;
  }

  .brand-desc {
    font-size: 0.875rem;
  }

  .auth-card {
    padding: 32px 24px;
    border-radius: 20px;
  }

  .auth-title {
    font-size: 1.5rem;
  }
}
</style>
