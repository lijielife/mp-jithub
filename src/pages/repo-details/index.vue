<template>
    <div class="repo-details-page">
        <div class="common-flex repo-summary">
            <div class="summary-item repo-owner-avatar">
                <img v-show="loaded && repoInfo['avatar_url']" :src="repoInfo['avatar_url']" @load="load">
                <img v-if="!loaded" src="/static/images/avatar-default.png">
            </div>
            <div class="summary-item repo-name">
                {{repoInfo.name ? repoInfo.name: 'N/A'}}
            </div>
            <div class="summary-item repo-description">
                {{repoInfo.description ? repoInfo.description : 'N/A'}}
            </div>
        </div>
        <div class="common-flex repo-meta-data">
            <div class="repo-meta">
                <span class="repo-stars">{{repoInfo['stargazers_count'] ? repoInfo['stargazers_count'] : 0}}</span>
                <span>Stargazers</span>
            </div>
            <div class="repo-meta">
                <span class="repo-watchers">{{repoInfo['subscribers_count'] ? repoInfo['subscribers_count'] : 0}}</span>
                <span>Watchers</span>
            </div>
            <div class="repo-meta">
                <span class="repo-forks">{{repoInfo['forks'] ? repoInfo['forks'] : 0}}</span>
                <span>Forks</span>
            </div>
        </div>
        <div class="repo-basic-info">
            <div class="common-flex">
                <span class="flex-item">
                    <img src="/static/images/public.png">
                    {{repoInfo.private ? 'Private': 'Public'}}
                </span>
                <span class="flex-item">
                    <img src="/static/images/lang.png">
                    {{repoInfo.language ? repoInfo.language: 'N/A'}}
                </span>
            </div>
            <div class="common-flex">
                 <span class="flex-item">
                    <img src="/static/images/issue.png">
                    {{repoInfo['open_issues'] ? repoInfo['open_issues']: 0}} Issues
                </span>
                <span class="flex-item">
                    <img src="/static/images/fork.png">
                    {{branches.length}} {{branches.length > 1 ? 'Branches' : 'Branch'}}
                </span>
            </div>
            <div class="common-flex">
                <span class="flex-item">
                    <img src="/static/images/date.png">
                    {{repoInfo['created_date'] ? repoInfo['created_date'] : 'N/A'}}
                </span>
                <span class="flex-item">
                    <img src="/static/images/tool.png">
                    {{repoInfo['format_size'] ? repoInfo['format_size'] : 'N/A'}}
                </span>
            </div>
            <div class="common-flex owner" @click="enterUserDetail">
                <span class="flex-item">
                    <img src="/static/images/owner.png">
                    Owner
                    <span class="more-info">
                        {{repoInfo['owner'] ? repoInfo['owner'].login: 'N/A'}}
                        <i class="right-arrow"></i>
                    </span>
                </span>
            </div>
        </div>
        <loading v-if="loading"></loading>
    </div>
</template>

<script>
    import './index.less';
    import { mapActions, mapGetters } from 'vuex';

    import loading from '@src/components/loading/index.vue';
    // #365
    // import rightArrow from '@src/components/right-arrow/index.vue';

    export default {
        data () {
            return {
                userInfo: {},
                loaded: false,
                loading: false,
                repo: {}
            };
        },

        components: {
            loading
        },

        computed: {
            ...mapGetters({
                loading: 'getLoadingState',
                repoInfo: 'getRepoDetailState',
                branches: 'getBranches'
            })
        },

        methods: {
            ...mapActions([
                'getRepoDetailInfo',
                'getRepoBranches'
            ]),
            getUserInfo () {
                // 调用登录接口
                wx.login({
                    success: () => {
                        wx.getUserInfo({
                            success: (res) => {
                                this.userInfo = res.userInfo;
                            }
                        });
                    }
                });
            },

            load () {
                this.loaded = true;
            },

            resetLoading () {
                this.loading = false;
            },  

            enterUserDetail () {
                console.log('user detail');
            }
        },

        onShareAppMessage () {
            return {
                title: `${this.repo.username} / ${this.repo.reponame}`,
                desc: `${this.repo.description}`,
                imageUrl: '',
                path: '/pages/repo-details/repo-details'
            };
        },

        onShow () {
            try {
                const repo = wx.getStorageSync('repo-detail');
                if (!repo) {
                    throw new Error();
                }
                this.repo = { ...JSON.parse(repo) };
            } catch (e) {
                wx.showModal({
                    title: '',
                    content: '可能出现了一些错误, 请稍后再试',
                    showCancel: false,
                    confirmText: '我知道了'
                });
                return;
            }
            this.loading = true;
            wx.setNavigationBarTitle({
                title: `${this.repo.username} / ${this.repo.reponame}` || 'Jithub'
            });

            this.getRepoDetailInfo(this.repo).then(this.resetLoading).catch(this.resetLoading);
            this.getRepoBranches(this.repo).then(this.resetLoading).catch(this.resetLoading);
        },

        mounted () {
            this.getUserInfo();
        }
    };
</script>