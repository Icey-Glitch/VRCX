<template>
    <el-dialog
        class="x-dialog"
        v-model="chatboxBlacklistDialog.visible"
        :title="t('dialog.chatbox_blacklist.header')"
        width="600px">
        <div v-if="chatboxBlacklistDialog.visible" v-loading="chatboxBlacklistDialog.loading">
            <h2>{{ t('dialog.chatbox_blacklist.keyword_blacklist') }}</h2>
            <el-input
                v-for="(item, index) in chatboxBlacklist"
                :key="index"
                v-model="chatboxBlacklist[index]"
                size="small"
                style="margin-top: 5px"
                @change="saveChatboxBlacklist">
                <template #append>
                    <el-button
                        :icon="Delete"
                        @click="
                            chatboxBlacklist.splice(index, 1);
                            saveChatboxBlacklist();
                        ">
                    </el-button>
                </template>
            </el-input>
            <el-button size="small" style="margin-top: 5px" @click="chatboxBlacklist.push('')">
                {{ t('dialog.chatbox_blacklist.add_item') }}
            </el-button>
            <br />
            <h2>{{ t('dialog.chatbox_blacklist.user_blacklist') }}</h2>
            <el-tag
                v-for="user in chatboxUserBlacklist"
                :key="user[0]"
                type="info"
                disable-transitions
                style="margin-right: 5px; margin-top: 5px"
                closable
                @close="deleteChatboxUserBlacklist(user[0])">
                <span>{{ user[1] }}</span>
            </el-tag>
        </div>
    </el-dialog>
</template>

<script setup>
    import { Delete } from '@element-plus/icons-vue';
    import { ref } from 'vue';
    import { storeToRefs } from 'pinia';
    import { useI18n } from 'vue-i18n';

    import { usePhotonStore } from '../../../stores';

    import configRepository from '../../../service/config';

    const { t } = useI18n();
    const photonStore = usePhotonStore();
    const { chatboxUserBlacklist, chatboxBlacklist } = storeToRefs(photonStore);

    defineProps({
        chatboxBlacklistDialog: {
            type: Object,
            required: true
        }
    });

    const emit = defineEmits(['deleteChatboxUserBlacklist']);

    // Load blacklist from config on mount (if needed)
    initChatboxBlacklist();

    async function initChatboxBlacklist() {
        const stored = await configRepository.getString('VRCX_chatboxBlacklist');
        if (stored) {
            // Update the Pinia store array in-place for reactivity
            chatboxBlacklist.value.splice(0, chatboxBlacklist.value.length, ...JSON.parse(stored));
        }
    }

    async function saveChatboxBlacklist() {
        // Save the Pinia store array to config
        await configRepository.setString('VRCX_chatboxBlacklist', JSON.stringify(chatboxBlacklist.value));
        // Optionally, trigger any additional store update logic here if needed
    }

    function deleteChatboxUserBlacklist(userId) {
        emit('deleteChatboxUserBlacklist', userId);
    }
</script>
