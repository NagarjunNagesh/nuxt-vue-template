<template>
    <form class="extended-forms">
        <card footer-classes="text-left">
            <div slot="header">
                <h4 class="card-title">{{ $t('budget.add.title') }}</h4>
            </div>
            <div>
                <base-input :label="$t('budget.add.planned')" required v-model="model.planned"
                    v-validate="modelValidations.planned" :error="getError('planned')" name="planned" type="number"
                    autofocus :placeholder="currency">
                </base-input>

                <base-input :label="$t('budget.add.category')" required :error="getError('categoryId')"
                    name="categoryId">
                    <el-select v-model="model.categoryId" class="select-primary" name="categoryId"
                        v-validate="modelValidations.categoryId" :loading="loadingDataForSelect" :clearable="clearable"
                        autocomplete="on" :filterable="filterable">
                        <el-option v-for="category in categories" class="select-primary"
                            :label="getCategoryValue(category)" :value="category.sk" :key="category.sk">
                        </el-option>
                    </el-select>
                </base-input>

                <div class="small form-category">{{ $t('budget.add.required-fields') }}</div>
            </div>

            <template slot="footer">
                <base-button native-type="submit" @click.native.prevent="validate" type="primary">{{
                        $t('budget.add.submit')
                }}</base-button>
                <nuxt-link class="float-right" to="/budgets">{{
                        $t('budget.add.to')
                }}</nuxt-link>
            </template>
        </card>
    </form>
</template>
<script>
import { DatePicker, Select, Option } from 'element-ui';

export default {
    components: {
        [DatePicker.name]: DatePicker,
        [Select.name]: Select,
        [Option.name]: Option
    },
    data() {
        return {
            filterable: true,
            clearable: true,
            model: {
                planned: null,
                categoryId: null,
            },
            modelValidations: {
                planned: {
                    required: true,
                    min_value: 1,
                },
                categoryId: {
                    required: true
                }
            },
            categories: [],
            loadingDataForSelect: true,
            currency: null
        };
    },
    methods: {
        getError(fieldName) {
            return this.errors.first(fieldName);
        },
        getCategoryValue(category) {
            return category.category_type + " : " + category.category_name
        },
        async getCategories(userId) {
            await this.$axios.$post(process.env.api.categories, {
                user_id: userId,
            }).then((response) => {
                this.categories = response;
                // Change loading to false
                this.loadingDataForSelect = false
            }).catch((response) => {
                let errorMessage = this.$lastElement(this.$splitElement(response.data.errorMessage, ':'));
                this.$notify({ type: 'danger', icon: 'tim-icons icon-simple-remove', verticalAlign: 'bottom', horizontalAlign: 'center', message: errorMessage });
            });
        },
        async validate() {
            let wallet = await this.$wallet.setCurrentWallet(this);
            this.$validator.validateAll().then(isValid => {
                this.$emit('on-submit', this.model, isValid, wallet.WalletId);
            });
        },
    },
    async mounted() {
        // Fetch the current user ID
        let userId = this.$authentication.fetchCurrentUser(this).financialPortfolioId;
        // Fetch Data from API
        await this.getCategories(userId);
        // Wallet Currency
        let wallet = await this.$wallet.setCurrentWallet(this);
        this.currency = wallet.WalletCurrency;
    }
};
</script>
<style>
</style>
