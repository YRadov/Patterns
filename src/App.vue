<template>
    <div id="app">
        <div class="container">
            <h3 class="text-uppercase text-center text-danger">
                Category:
                <select v-model="category" v-on:change="changeCat">
                    <option v-for="option in options" :value="option.id">
                        {{ option.name }}
                    </option>
                </select>
            </h3>
            <h3 v-if="sub_cat_list.length > 0" class="text-uppercase text-center text-danger">
                Sub Cat:
                <select v-model="sub_category" v-on:change="changeSubCat">
                    <option v-for="sub_cat in sub_cat_list" :value="sub_cat.subtitle">
                        {{ sub_cat.subtitle}} {{ sub_cat.ru }}
                    </option>
                </select>
            </h3>

            <div class="jumbotron">
                <div class="num"
                     v-if="data.length > 0 && !current.subtitle"
                     v-bind:class="{ red: isEven, blue: !isEven }">
                    {{ wordCnt + 1 }}
                </div>
                <p class="expression red" v-if="current.subtitle">{{ current.subtitle }} Подраздел</p>
                <p class="expression" v-if="lang.ru"><b>{{ current.ru }}</b></p>
                <p class="expression" v-if="lang.en"><b>{{ current.en }}</b></p>
            </div><!-- .jumbotron -->

            <div class="flex m_top_10">
                <button @click="prevWord" type="button" class="btn btn-lg btn-default pull-left" style="flex-grow: 2;">
                    <i aria-hidden="true" class="fa fa-backward fa-2x green"></i>
                </button>
                <button @click="startWord" type="button" class="btn btn-lg btn-default" style="flex-grow:1;">
                    <i aria-hidden="true" class="fa fa-stop-circle-o fa-2x red"></i>
                </button>
                <button @click="nextWord" type="button" class="btn btn-lg btn-default pull-right" style="flex-grow: 2;">
                    <i aria-hidden="true" class="fa fa-forward fa-2x green"></i>
                </button>
            </div>
            <p v-if="data.length > 0" class="change_lang btn-group">
                <button @click="switchLangRu" type="button" class="btn btn-lg"
                        :class="{'btn-danger': lang.ru, 'btn-success': !lang.ru}">
                    Russian
                </button>

                <button type="button" class="btn btn-default">Вкл/Выкл</button>

                <button @click="switchLangEn" type="button" class="btn btn-lg"
                        :class="{'btn-danger': lang.en, 'btn-success': !lang.en}">
                    English
                </button>
            </p>

            <!--********** TIMER **********-->
            <div v-if="timer_target" class="screen  m_top_40 text-center">
                Осталось: <span class="red">{{ timer }}</span> сек
            </div>
            <div class="flex timer">
                <button @click="start" :disabled="!play_enabled || play_on" type="button"
                        class="btn btn-lg btn-default pull-left" style="flex-grow: 2;">
                    <i aria-hidden="true" class="fa fa-play fa-2x green"></i>
                </button>
                <button @click="pause" :disabled="!play_on || pause_on" type="button" class="btn btn-lg btn-default"
                        style="flex-grow:2;">
                    <i aria-hidden="true" class="fa fa-pause fa-2x"></i>
                </button>
                <button @click="reset" :disabled="!play_enabled" type="button" class="btn btn-lg btn-default pull-right"
                        style="flex-grow: 2;">
                    <i aria-hidden="true" class="fa fa-stop fa-2x red"></i>
                </button>
            </div>
            <div class="form-group m_top_40">
                <h2>
                    <label>
                        <input @keyup="setTimerTarget" type="number" min="0" v-model.number="timer_target"
                               placeholder="0"> Интервал, сек
                    </label>
                </h2>
            </div>

        </div><!-- .container -->

    </div><!-- #app -->
</template>

<script>
    import _ from 'underscore';

    export default {
        name: 'app',
        data() {
            return {
                lang: {
                    ru: true,
                    en: false
                },
                wordCnt: 0,
                options: Category,
                category: 0,
                sub_category: 0,
                data: [],
                current: {
                    ru: "",
                    en: ""
                },
                timer_target: '',
                timer: 0,
                play_on: false,
                pause_enabled: false,
                pause_on: false,
                interval: null,
            }
        },
        mounted() {
            this.getSavedState();
        },
        computed: {
            play_enabled() {
                return (this.category > 0 && this.timer_target > 0);
            },
            WordsLength() {
                return this.data.length;
            },
            isEven() {
                return !(this.wordCnt % 2);
            },
            sub_cat_list() {
                return _.filter(this.data, function (item) {
                    return item.hasOwnProperty('subtitle');
                });
            }
        },
        methods: {
            getSavedState() {
                let category = parseInt(localStorage.getItem('category'));
                let current_word_id = parseInt(localStorage.getItem('current_word_id'));
                if (category) {
                    this.category = category;
                    this.changeCat();
                    this.wordCnt = current_word_id;
                    this.changeWord();
                }
//                console.log('category => ' + category + '\nthis.category => ' + this.category);
//                console.log('current_word_id => ' + current_word_id + '\nthis.wordCnt => ' + this.wordCnt);
            },
            switchLangEn: function () {
                this.lang.en = !this.lang.en;
            },
            switchLangRu: function () {
                this.lang.ru = !this.lang.ru;
            },
            prevWord: function () {
                this.wordCnt--;
                if (this.wordCnt < 0) {
                    this.wordCnt = this.WordsLength - 1;
                }
                this.changeWord();
            },
            nextWord: function () {
                this.wordCnt++;
                if (this.wordCnt >= this.WordsLength) {
                    this.wordCnt = 0;
                }
                this.changeWord();
            },
            startWord: function () {
                this.wordCnt = 0;
                this.changeWord();
            },
            changeWord: function (init = false) {
                this.current = this.data[this.wordCnt];
                // save current word to storage
                localStorage.setItem('current_word_id', this.wordCnt);
            },
            changeCat: function (event) {
                this.reset();
                this.data = _.where(Data, {category: this.category});
                this.wordCnt = 0;
                this.changeWord();
                this.lang = {
                    ru: true,
                    en: false
                };
                // save category to storage
                localStorage.setItem('category', this.category);
            },
            changeSubCat: function (event) {
                // найти id начала нового раздела
                let index;
                for (let i = 0; i < this.data.length; i++) {
                    if (this.data[i].subtitle == this.sub_category) {
                        index = i;
                        break;
                    }
                }
                // текущее слово в положение начала раздела
                this.wordCnt = index;
                this.changeWord();
                this.lang = {
                    ru: true,
                    en: false
                };
            },
            start() {
                // выключить паузу
                this.pause_on = false;
                // включить play
                this.play_on = true;
                this.interval = setInterval(() => this.timerStep(), 1000)
            },
            pause() {
                // включить паузу
                this.pause_on = true;
                // выключить play
                this.play_on = false;
                // остановить таймер
                clearInterval(this.interval);
            },
            reset() {
                // выключить паузу
                this.pause_on = false;
                // выключить play
                this.play_on = false;
                // остановить таймер
                clearInterval(this.interval);
                // назначить новый timer
                this.timer = this.timer_target;
            },
            timerStep() {
                this.timer--;
                if (this.timer == 0) {
                    this.nextWord();
                }
                if (this.timer < 1) {
                    this.timer = this.timer_target;
                }
            },
            setTimerTarget() {
                // остановить и сбросить все в ноль(но позицию слова оставить)
                this.reset();
                // назначить новый timer
                this.timer = this.timer_target;
            },
        }

    }
</script>

<style lang="scss">
    .screen {
        font-size: 3em;
    }

    .jumbotron {
        position: relative;
        min-height: 350px;
        padding: 40px 10px !important;
        margin-top: 30px;
        .num {
            position: absolute;
            left: 10px;
            top: 10px;
            font-size: 20px;
        }
    }

    //.jumbotron

    .expression {
        font-size: 3em !important;
    }

    .flex {
        display: flex;
        justify-content: space-around;
        height: 120px;
    }

    .m_top_10 {
        margin-top: 10px;
    }

    .m_top_40 {
        margin-top: 40px;
    }

    .change_lang {
        margin-top: 40px;
        .btn {
            height: 100px;
            font-size: 2em;
            padding: 0 50px;
        }
    }

    //#change_lang

    .timer {
        margin-top: 40px;
    }

    //.timer

    #app {
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
        margin-top: 60px;
    }

    h1, h2 {
        font-weight: normal;
    }

    ul {
        list-style-type: none;
        padding: 0;
    }

    li {
        display: inline-block;
        margin: 0 10px;
    }

    a {
        color: #42b983;
    }

    .red {
        color: red;
    }

    .green {
        color: green;
    }

    .blue {
        color: blue;
    }
</style>
