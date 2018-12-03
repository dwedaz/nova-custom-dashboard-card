<template>
    <div>
        <div style="display:inline-block;">
        <button @click="addCardToLayout" class="mb-3 btn btn-default btn-primary">Add Card</button>
        <v-select multiple v-model="selectedCard" :options="options" style="width:300px;display:inline-block;background-color:white;" class="mb-3" ></v-select>
        </div>
        

        <div class="-mx-3 mb-3">

            <grid-layout
                    :layout="layout"
                    :col-num="12"
                    :row-height="30"
                    :is-draggable="true"
                    :is-resizable="true"
                    :is-mirrored="false"
                    :vertical-compact="true"
                    :margin="[0, 0]"
                    :use-css-transforms="true"
                    @layout-updated="saveLayout">
                <grid-item v-for="(item, index) in layout"
                           v-if="isComponentAvailable(item.card.component)"
                           :key="item.i"
                           :x="item.x"
                           :y="item.y"
                           :w="item.w"
                           :h="item.h"
                           :i="item.i"
                >
                    <div class="h-full">
                        <button
                                @click="removeCard(index)"
                                title="Delete"
                                class="absolute pin-r pin-b appearance-none mr-3 text-70 hover:text-primary"
                        >
                            <icon type="delete" />
                        </button>
                        <dynamic-card-wrapper
                                class="pb-8"
                                :card="item.card"
                                :size="'large'"
                                :key="`${item.card.component}.${item.card.name}`"
                        />
                    </div>
                </grid-item>
            </grid-layout>
        </div>
    </div>
</template>

<script>

import VueGridLayout from 'vue-grid-layout'

import DynamicCardWrapper from './DynamicCardWrapper'
import vSelect from 'vue-select'




const GridLayout = VueGridLayout.GridLayout;
const GridItem = VueGridLayout.GridItem;

export default {
    props: ['card'],
    components: {
        DynamicCardWrapper,
        GridLayout,
        GridItem,
        vSelect
    },
    computed: {
      options: function () {
        var data = [];
        for (var i=0; i< this.cards.length; i++){
          data.push( { value: this.cards[i], label: this.cards[i].name ||this.cards[i]['card-name'] || this.cards[i].component} );
        }
        return data;
      }
    },
    data() {
        return {
            modalOpen: false,
            selectedCard: null,
            cards: [],
            layout: [],
            item: {
              value: '',
              text: ''
            }
        };
    },

    methods: {
        removeCard(index) {
            this.layout.splice(index, 1);
            this.saveLayout(this.layout);
        },

        isComponentAvailable(component) {
            return typeof this.$options.components[component] !== 'undefined';
        },

        saveLayout(layout) {
            localStorage.setItem('nova-dashboard', JSON.stringify(layout));
        },

        openModal() {
            this.modalOpen = true;
        },

        closeModal() {
            this.modalOpen = false;
        },

        addCardToLayout(){
            var add = null;
            for(var j=0; j<this.selectedCard.length; j++ ){
                add = true;
                let card_id = this.selectedCard[j].value.component;
                let card = {
                    x: 0,
                    y: 0,
                    w: 12,
                    h: 4,
                    i: card_id,
                    card: this.selectedCard[j].value
                };

              
                for(var i=0 ;i< this.layout.length; i++){
                  if (this.layout[i].card.component == card.card.component ){
                    alert("Cant add same card ("+card.card.component+")") 
                    add = false;
                  }
                }
                if (add){
                     this.layout.push(card);
                }
               

                this.saveLayout(this.layout);

               
            }
             this.selectedCard = null;
           
        },

        async fetchCards() {
            const { data: cards } = await Nova.request().get('/nova-vendor/beyondcode/nova-custom-dashboard-card/cards');

            this.cards = cards;

            let layout = localStorage.getItem('nova-dashboard');
            if (layout) {
                this.layout = JSON.parse(layout);
            }
        },
    },

    mounted() {
        this.fetchCards();
    },
}
</script>

<style>

    .vue-resizable-handle {
        z-index: 5000;
        position: absolute;
        width: 20px;
        height: 20px;
        bottom: 0;
        right: 0;
        background: url('data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBzdGFuZGFsb25lPSJubyI/Pg08IS0tIEdlbmVyYXRvcjogQWRvYmUgRmlyZXdvcmtzIENTNiwgRXhwb3J0IFNWRyBFeHRlbnNpb24gYnkgQWFyb24gQmVhbGwgKGh0dHA6Ly9maXJld29ya3MuYWJlYWxsLmNvbSkgLiBWZXJzaW9uOiAwLjYuMSAgLS0+DTwhRE9DVFlQRSBzdmcgUFVCTElDICItLy9XM0MvL0RURCBTVkcgMS4xLy9FTiIgImh0dHA6Ly93d3cudzMub3JnL0dyYXBoaWNzL1NWRy8xLjEvRFREL3N2ZzExLmR0ZCI+DTxzdmcgaWQ9IlVudGl0bGVkLVBhZ2UlMjAxIiB2aWV3Qm94PSIwIDAgNiA2IiBzdHlsZT0iYmFja2dyb3VuZC1jb2xvcjojZmZmZmZmMDAiIHZlcnNpb249IjEuMSINCXhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgeG1sbnM6eGxpbms9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkveGxpbmsiIHhtbDpzcGFjZT0icHJlc2VydmUiDQl4PSIwcHgiIHk9IjBweCIgd2lkdGg9IjZweCIgaGVpZ2h0PSI2cHgiDT4NCTxnIG9wYWNpdHk9IjAuMzAyIj4NCQk8cGF0aCBkPSJNIDYgNiBMIDAgNiBMIDAgNC4yIEwgNCA0LjIgTCA0LjIgNC4yIEwgNC4yIDAgTCA2IDAgTCA2IDYgTCA2IDYgWiIgZmlsbD0iIzAwMDAwMCIvPg0JPC9nPg08L3N2Zz4=');
        background-position: bottom right;
        padding: 0 3px 3px 0;
        background-repeat: no-repeat;
        background-origin: content-box;
        box-sizing: border-box;
        cursor: se-resize;
    }

    .vue-grid-item.resizing {
        opacity: 0.9;
    }

    .vue-grid-item.static {
        background: #cce;
    }

    .vue-grid-item .text {
        font-size: 24px;
        text-align: center;
        position: absolute;
        top: 0;
        bottom: 0;
        left: 0;
        right: 0;
        margin: auto;
        height: 100%;
        width: 100%;
    }

    .vue-grid-item .no-drag {
        height: 100%;
        width: 100%;
    }

    .vue-draggable-handle {
        position: absolute;
        width: 20px;
        height: 20px;
        top: 0;
        left: 0;
        background: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='10' height='10'><circle cx='5' cy='5' r='5' fill='#999999'/></svg>") no-repeat;
        background-position: bottom right;
        padding: 0 8px 8px 0;
        background-repeat: no-repeat;
        background-origin: content-box;
        box-sizing: border-box;
        cursor: pointer;
    }
</style>