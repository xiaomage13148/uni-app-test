<template>
    <view>
        <page-head :title="title"></page-head>
        <view class="uni-padding-wrap uni-common-mt">
            <button type="primary">{{ content }}</button>
            <button type="primary"
                    @click="stopPolling">清除定时器
            </button>
        </view>
    </view>
</template>
<script>
export default {
    data() {
        return {
            title: 'button',
            content: '',
            timer: null
        };
    },
    async onLoad(option) {
        const id = option.id;
        console.log('---->id', id);
        console.log('---->startPolling before');
        await this.startPolling();
        console.log('---->startPolling after');
    },
    onShow() {
    },
    onHide() {
        this.stopPolling();
    },
    onUnload() {
        this.stopPolling();
    },
    methods: {
        // 异步请求
        syncPromise() {
            const self = this;
            let info = {
                code: -1,
                msg: ''
            };
            return new Promise(resolve => {
                uni.request({
                    url: 'http://localhost:8099/sys/user/getAllUser',
                    header: {'token': 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MTE5NDA3MDgsImlhdCI6MTcxMTg5NzUwOCwidXNlcm5hbWUiOiJ4aWFvbWEifQ.vUj6mxtsSYciJ_afDP4wvordqXz-G9C6A_9kpMd4w_w'},
                    method: 'GET',
                    timeout: 3000,
                    success: ({data}) => {
                        if (data.code !== '00000') {
                            info.code = 3;
                            info.msg = data.msg;
                        } else if (data.data.length === 3) {
                            info.code = 2;
                            info.msg = '数据没变化';
                        } else {
                            info.code = 1;
                            info.msg = '数据出现变化';
                        }
                        console.log('---->syncPromise,info', info);
                        resolve(info);
                    },
                    fail: err => {
                        info.code = 3;
                        info.msg = err.message;
                        console.log('---->syncPromise,info', info);
                        resolve(info);
                    }
                });
            });
        },

        // 轮询
        pollingPromiseZ() {
            const self = this;
            return new Promise(resolve => {
                self.timer = setTimeout(async () => {
                    const result = await self.syncPromise();
                    resolve(result);
                }, 3000);
            });
        },

        // 开始轮询逻辑
        async startPolling() {
            console.log('---->开始执行startPolling');
            const self = this;
            // 先清除定时器
            self.stopPolling();
            // 立刻执行一次异步请求
            let pollingInfo = await self.syncPromise();

            // 根据异步请求结果，判断是否需要开启计时轮询
            while (pollingInfo.code === 2) {
                pollingInfo = await self.pollingPromiseZ();
            }
            // 若异步请求被 clearTimeout(timer)，这里不会被执行打印输出。
            console.log('轮询请求处理完成！');
            console.log('---->pollingInfo', pollingInfo);

            if (pollingInfo.code === 1) {
                uni.showToast({
                    icon: 'none',
                    title: pollingInfo.msg
                });
            } else if (pollingInfo.code === 3) {
                // 跳转回之前的界面
                uni.navigateBack();
            }
        },

        // 清除定时器
        stopPolling() {
            if (this.timer !== null) {
                clearTimeout(this.timer);
                this.timer = null;
            }
        }
    }
};
</script>

<style>
button {
    margin-top: 30 rpx;
    margin-bottom: 30 rpx;
}

.button-sp-area {
    margin: 0 auto;
    width: 60%;
}

.mini-btn {
    margin-right: 10 rpx;
}
</style>
