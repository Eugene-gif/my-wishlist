<script setup lang="ts">
definePageMeta({ middleware: 'auth' })
useSeoMeta({ title: 'Мои вишлисты' })

const { user } = useUserSession()
const UIname = computed(() => user.value?.name || 'Неизвестный')

const toast = useToast()
const { data: wishlists, status, refresh } = await useFetch('/api/wishlists')

const state = reactive({ title: '' })

const create = async () => {
  try {
    await $fetch('/api/wishlists', { method: 'POST', body: state })
    state.title = ''
    await refresh()
  }
  catch (error) {
    const message = (error as { data?: { message?: string } }).data?.message
    toast.add({ title: message ?? 'Не удалось создать вишлист', color: 'error' })
  }
}
</script>

<template>
  <UContainer>
    <h1 class="text-2xl font-semibold">
      Мои вишлисты
    </h1>
    <p class="mt-1 text-muted">
      Привет, {{ UIname }}
    </p>

    <UForm
      :schema="wishlistSchema"
      :state="state"
      class="mt-6 flex items-start gap-2"
      @submit="create"
    >
      <UFormField
        name="title"
        class="flex-1"
      >
        <UInput
          v-model="state.title"
          placeholder="Например, День рождения"
          class="w-full"
        />
      </UFormField>
      <UButton
        type="submit"
        loading-auto
        icon="i-lucide-plus"
        label="Создать"
      />
    </UForm>

    <p
      v-if="status === 'pending'"
      class="mt-8 text-muted"
      role="status"
    >
      Загружаем вишлисты…
    </p>
    <UEmpty
      v-else-if="status === 'error'"
      class="mt-8"
      icon="i-lucide-triangle-alert"
      title="Не удалось загрузить вишлисты"
    >
      <template #actions>
        <UButton
          label="Повторить"
          @click="refresh()"
        />
      </template>
    </UEmpty>
    <UEmpty
      v-else-if="!wishlists?.length"
      class="mt-8"
      icon="i-lucide-gift"
      title="Здесь пока пусто"
      description="Скоро здесь появятся ваши вишлисты"
    />
    <ul
      v-else
      class="mt-6 space-y-2"
    >
      <li
        v-for="wishlist in wishlists"
        :key="wishlist.id"
      >
        <UCard>{{ wishlist.title }}</UCard>
      </li>
    </ul>
  </UContainer>
</template>
