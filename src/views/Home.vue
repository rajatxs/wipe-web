<script setup>
import { toRaw, watch, computed } from 'vue';
import { useRouter } from 'vue-router';
import { useQuery, useMutation, useQueryClient } from '@tanstack/vue-query';
import { fetchSubscriptions, changeSubscriptionStatus } from '../api/subs';
import { Subscription } from '../models/Subscription';
import { avatarUrl } from '../utils/common';
import appState from '../state';
import NotificationIcon from '../icons/notification.vue';
import LoaderView from '../components/LoaderView.vue';
import SubscriptionStatus from '../components/SubscriptionStatus.vue';
import Switch from '../components/Switch.vue';
import store from '../utils/store';

const client = useQueryClient();
const router = useRouter();
const {
    isLoading,
    data: subs,
    isFetchedAfterMount,
    refetch,
    error,
} = useQuery({
    enabled: true,
    queryKey: ['subs'],
    queryFn: fetchSubscriptions,
    refetchOnMount: true,
    initialData: store.getSubscriptions(),
    refetchInterval: 5000,
    refetchOnWindowFocus: false,
    refetchOnReconnect: true,
    refetchIntervalInBackground: false,
});

const toggleStatusMutation = useMutation({
    mutationFn: changeSubscriptionStatus,
    onSuccess: refetch,
    onError() {
        alert('Unable to update subscription status');
    },
});

const presenceSubs = computed(function () {
    if (Array.isArray(subs.value)) {
        return subs.value
            .filter((sub) => sub.event === 'presence.update')
            .filter(function (sub) {
                if (appState.viewAllSubscriptions) {
                    return sub;
                } else {
                    return sub.enabled;
                }
            });
    } else {
        return [];
    }
});

const contactSubs = computed(function () {
    if (Array.isArray(subs.value)) {
        return subs.value.filter((sub) => sub.event === 'contacts.update');
    } else {
        return [];
    }
});

/**
 * Handles the view presence action
 * @param {Subscription} sub
 */
function handleViewPresence(sub) {
    const rawSub = toRaw(sub);
    client.setQueryData(['subs', rawSub.id], rawSub);
    router.push(`/subs/${rawSub.id}`);
}

watch(isFetchedAfterMount, () => {
    const raw = toRaw(subs.value);
    store.setSubscriptions(raw);
});
</script>

<template>
    <LoaderView v-if="isLoading" message="Loading" />
    <div v-else-if="subs" class="flex flex-col py-5 w-full gap-2">
        <!-- Presence subscription -->
        <div v-for="sub in presenceSubs" :key="sub.id" class="subs-item">
            <div
                role="button"
                class="subs-router-link"
                @click="handleViewPresence(sub)">
                <div class="relative w-12 h-12">
                    <img
                        :src="avatarUrl(sub.alias)"
                        :alt="sub.alias"
                        :title="sub.alias"
                        width="46"
                        height="46"
                        class="rounded-full" />
                    <NotificationIcon v-if="sub.notify" class="subs-alert-badge" />
                </div>
                <div class="flex flex-col gap-1">
                    <div class="font-medium leading-5">{{ sub.alias }}</div>
                    <SubscriptionStatus :show="sub.enabled" :sub-id="sub.id" :fallback="sub.phone" />
                </div>
            </div>
            <div class="w-[20%] flex items-center justify-end pr-5 h-12">
                <Switch
                    :id="`sub-switch-${sub.id}`"
                    :value="sub.enabled"
                    @value="toggleStatusMutation.mutate({ id: sub.id, enabled: $event })" />
            </div>
        </div>

        <template v-if="appState.viewAllSubscriptions">
            <hr class="w-[92%] mx-auto my-2 border-neutral-200 dark:border-neutral-700" />

            <!-- Contact subscription -->
            <div v-for="sub in contactSubs" :key="sub.id" class="subs-item">
                <div
                    role="button"
                    class="subs-router-link"
                    @click="handleViewPresence(sub)">
                    <div class="relative w-12 h-12">
                        <img
                            :src="avatarUrl(sub.alias)"
                            :alt="sub.alias"
                            :title="sub.alias"
                            width="46"
                            height="46"
                            class="rounded-full" />
                        <NotificationIcon v-if="sub.notify" class="subs-alert-badge" />
                    </div>
                    <div class="flex flex-col gap-1">
                        <div class="font-medium leading-5">{{ sub.alias }}</div>
                        <div class="text-xs text-neutral-500">Contact</div>
                    </div>
                </div>
                <div class="w-[20%] flex items-center justify-end pr-5 h-12">
                    <Switch
                        :id="`sub-switch-${sub.id}`"
                        :value="sub.enabled"
                        @value="toggleStatusMutation.mutate({ id: sub.id, enabled: $event })" />
                </div>
            </div>
        </template>
    </div>
    <div v-else-if="error" class="h-3/5 flex flex-col items-center justify-center">
        <h3 class="font-medium text-neutral-500">Something went wrong</h3>
    </div>
    <div v-else class="h-3/5 flex flex-col items-center justify-center">
        <h3 class="font-medium text-neutral-500">No subscriptions</h3>
    </div>
</template>

<style>
.subs-item {
    @apply flex w-full h-16 justify-between items-center;
}

.subs-router-link {
    @apply flex items-center h-full gap-2 w-[80%] px-5 rounded-md transition-colors active:bg-neutral-100 md:hover:bg-neutral-100 dark:active:bg-neutral-800 md:dark:hover:bg-neutral-800;
}

.subs-alert-badge {
    @apply absolute w-5 h-5 top-[-3px] right-[-3px] p-1 rounded-full bg-neutral-200 fill-neutral-500 dark:bg-neutral-700 dark:fill-white;
}
</style>
