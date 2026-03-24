<template>
    <AuthLayout
        branding-title="Short Link Service"
        branding-description="专业的短链接生成与管理平台，提供详细的数据分析和稳定的访问服务。"
    >
        <div class="text-center lg:text-left">
            <h2 class="text-3xl font-bold tracking-tight text-gray-900 dark:text-gray-100">
                欢迎回来
            </h2>
            <p class="mt-2 text-gray-500 dark:text-gray-400">请输入您的账号信息以登录控制台</p>
        </div>

        <a-form
            :model="form"
            layout="vertical"
            @submit="handleEmailLogin"
            class="mt-8"
        >
            <a-form-item
                field="email"
                label="邮箱"
                :rules="[
                    { required: true, message: '请输入邮箱' },
                    { type: 'email', message: '请输入有效的邮箱地址' },
                ]"
            >
                <a-input
                    v-model="form.email"
                    placeholder="name@example.com"
                    allow-clear
                    size="large"
                    class="rounded-lg!"
                >
                    <template #prefix>
                        <icon-email class="text-gray-400 dark:text-gray-500" />
                    </template>
                </a-input>
            </a-form-item>

            <a-form-item
                field="password"
                label="密码"
                :rules="[{ required: true, message: '请输入密码' }]"
            >
                <a-input-password
                    v-model="form.password"
                    placeholder="请输入密码"
                    allow-clear
                    size="large"
                    class="rounded-lg!"
                >
                    <template #prefix>
                        <icon-lock class="text-gray-400 dark:text-gray-500" />
                    </template>
                </a-input-password>
            </a-form-item>

            <div class="flex items-center justify-between mb-6">
                <a-checkbox>记住我</a-checkbox>
                <a-link
                    @click="handleForgotPassword"
                    class="text-sm font-medium"
                    >忘记密码?</a-link
                >
            </div>

            <a-button
                type="primary"
                html-type="submit"
                long
                size="large"
                :loading="userStore.isLoading"
                class="rounded-lg! h-12! text-base! font-medium!"
            >
                登录
            </a-button>
        </a-form>

        <div class="text-center mt-8">
            <a-link
                @click="goToHome"
                class="text-gray-400! hover:text-gray-600! dark:text-gray-500! dark:hover:text-gray-300! text-sm"
            >
                <icon-left /> 返回首页
            </a-link>
        </div>
    </AuthLayout>
</template>

<script setup lang="ts">
import { Message } from "@arco-design/web-vue";
import { IconEmail, IconLeft, IconLock } from "@arco-design/web-vue/es/icon";
import { reactive } from "vue";
import { useRouter } from "vue-router";
import AuthLayout from "@/components/AuthLayout.vue";
import { useUserStore } from "@/stores";

const router = useRouter();
const userStore = useUserStore();

const form = reactive({
	email: "",
	password: "",
});

// 处理邮箱登录
async function handleEmailLogin({ errors }: any) {
	if (errors) return;

	try {
		await userStore.loginWithEmail(form.email, form.password);
		Message.success("登录成功！");
		setTimeout(() => {
			router.push("/dashboard");
		}, 500);
	} catch (error: any) {
		console.error("登录错误:", error);

		// 处理不同的错误情况
		if (error.code === "USER_BANNED") {
			// 用户被禁用的特殊提示
			Message.error({
				content: "您的账号已被管理员禁用，如有疑问请联系管理员",
				duration: 5000,
			});
		} else if (error.message.includes("Invalid login credentials")) {
			Message.error("邮箱或密码错误");
		} else if (error.message.includes("Email not confirmed")) {
			Message.error("请先验证您的邮箱");
		} else if (error.message.includes("禁用")) {
			// 兜底处理所有包含"禁用"的错误消息
			Message.error({
				content: error.message,
				duration: 5000,
			});
		} else {
			Message.error(error.message || "登录失败，请稍后再试");
		}
	}
}

function handleForgotPassword() {
	router.push("/forgot-password");
}

function goToHome() {
	router.push("/");
}
</script>
