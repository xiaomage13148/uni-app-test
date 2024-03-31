<template>
    <view>
        <page-head :title="title"></page-head>
        <view class="uni-padding-wrap uni-common-mt">
            <button type="primary">{{ content }}</button>
        </view>
    </view>
</template>
<script>
export default {
    data() {
        return {
            title: 'button',
            content: '',
            index: 1,
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
        // 模拟异步请求
        syncPromise() {
            const self = this;
            return new Promise(resolve => {
                setTimeout(() => {
                    uni.showToast({
                        icon: 'none',
                        title: `第 ${self.index} 次请求`
                    });
                    console.log(`第 ${self.index} 次请求`);
                    resolve(self.index < 5 ? true : false);
                    self.index++;
                }, 200);
            });
        },

        // 轮询
        pollingPromiseZ() {
            const self = this;
            return new Promise(resolve => {
                self.timer = setTimeout(async () => {
                    const result = await self.syncPromise();
                    resolve(result);
                }, 2000);
            });
        },

        // 开始轮询逻辑
        async startPolling() {
            console.log('---->开始执行startPolling');
            const self = this;
            // 先清除定时器
            self.stopPolling();
            self.index = 1;
            // 立刻执行一次异步请求
            let needPolling = await self.syncPromise();

            console.log('---->needPolling', needPolling);

            // 根据异步请求结果，判断是否需要开启计时轮询
            while (needPolling) {
                needPolling = await self.pollingPromiseZ();
            }
            // 若异步请求被 clearTimeout(timer)，这里不会被执行打印输出。
            console.log('轮询请求处理完成！');
            // 跳转回之前的界面
            uni.navigateBack();
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
