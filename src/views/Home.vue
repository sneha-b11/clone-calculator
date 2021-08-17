<template>
  <div>
    
    <div class="outer_container">
      <div class="inner_container">
        <div>
          <h3>Selling Fee Calculator</h3>
          <p>Calculate the profit you can earn with your product</p>
          <model-select :options="options" v-model="item" placeholder="Select product"></model-select>
          <p>{{item.marketfee_percent}}%</p>
          <p>Enter the selling price of your product</p>
        </div>
        <form action="">
          <input v-model="selling_price" type="text" placeholder="Selling price (in rupees)" style="width: 220px; border:none; border-radius:5px; height: 30px;">
          <span class="first_button"> 
            <button type="button" class="btn btn-outline-primary btn-sm" style="border-radius: 20px;" @click="calculate_fee(selling_price)">Calculate Fees</button>
          <p v-if="this.errortext" class="text-danger">
            {{this.errortext}}
          </p>
          <p>Prices and rates shown below are indicative</p>
          </span>
          
        </form>
      </div>
      <div v-if="this.show_fee">
        <hr class="new1">
        <div style="display:flex">
            <p style="margin-left:20px">MarketPlace Fee ({{item.marketfee_percent}}%)</p>
            <p style="position: absolute; right: 60px;">Rs {{parseFloat(this.market_fee).toFixed(2)}}</p>
        </div>
        <div style="display: flex">
          <p style="margin-left:20px">Closing Fee</p>
          <p style="position: absolute; right: 60px;">Rs {{this.closing_fees.charges}}</p>
        </div>
            
        <div v-if="this.show_shipping">
          <div class="button">
            <button @click="with_shipping()" type="button" class="btn btn-outline-primary" id="second_button" style="border-radius: 20px;">With Shipping Fee</button>
              <button @click="without_shipping()" type="button" class="btn btn-outline-primary" id="third_button" style="border-radius: 20px;">Without Shipping Fee</button>
          </div>
          <div v-if="this.show_with_shipping">
            <p class="newP2">
              Shipping fees varies based on distance the product is shipped:
            </p>
            <div class="button">
              <button type="button" @click="selected_category('city')" class="btn btn-outline-primary" style="border-radius: 20px; margin:8px;">Within City</button>
              <button type="button" @click="selected_category('state')" class="btn btn-outline-primary" style="border-radius: 20px; margin:8px">Within State</button>
              <button type="button" @click="selected_category('rest_of_india')" class="btn btn-outline-primary" style="border-radius: 20px; margin:8px">Rest of India</button><br>
              <button type="button" @click="selected_category('metro')" class="btn btn-outline-primary" style="border-radius: 20px; margin:8px">Metro to Metro</button>
              <button type="button" @click="selected_category('special')" class="btn btn-outline-primary" style="border-radius: 20px; margin:8px">Special Regions</button>
            </div>
            <div>
              <p style="padding-top: 10px;">
                Enter the approximate weight of the product
              </p>
              <form action="">
                <input  type="text" v-model="weight" placeholder="Weight (in kgs)" style="width: 180px; border:none; border-radius:5px; height: 30px;">
                <button id="input_field" type="button" class="btn btn-outline-primary btn-sm" style="border-radius: 20px;" @click="submit()">Submit</button>
              </form>
            </div>
          </div>
        </div>
        <div v-if="show_total">
          <div style="display:flex">
            <p style="margin-left:20px">Shipping Fee</p>
            <p style="position:absolute; right:60px">Rs {{parseFloat(this.shipping_charges).toFixed(2)}}</p>
          </div>
          <div>
            <h6 class="shipping" v-if="show_with_shipping">({{this.weight}} kgs)</h6>
          </div>
          <hr>

          <div style="display:flex">
            <p style="margin-left:20px">Total Fee</p>
            <p style="position:absolute; right:60px">Rs {{total}}</p>
          </div>

          <div style="display:flex">
            <p style="margin-left:20px">GST on Total Fee (18%)</p>
            <p style="position:absolute; right:60px">Rs {{gst_total}}</p>
          </div>
          <hr>

          <div style="display:flex">
            <p style="margin-left:20px"><strong>Total Selling Fee </strong></p>
            <p style="position:absolute; right:60px"><strong>Rs {{grand_total}} </strong></p>
          </div>

          <div style="display:flex">
            <p style="margin-left:20px"><strong>Seller Profit</strong></p>
            <p style="position:absolute; right:60px"><strong>Rs {{profit}} </strong></p>
          </div>
          <button type="button" class="btn btn-outline-primary btn-sm" style="border-radius: 20px;" @click="back()">Back</button>
        </div>
      </div>
    </div> 
  </div>
</template>

<script>

import 'vue-search-select/dist/VueSearchSelect.css'
import { ModelSelect } from 'vue-search-select'
import 'vue-slick-carousel/dist/vue-slick-carousel.css'
// optional style for arrows & dots
import 'vue-slick-carousel/dist/vue-slick-carousel-theme.css'
import {auth, db} from '@/firebase/firebaseInit.js';
// import firebase from "firebase/app";

  export default {
  name: 'Home',
  components: {
    ModelSelect,
   },
  data() {
    return {
      showshippingtable:false,
      // arr : [
      //   {
      //     'weight' : 0.5,
      //     'city_price': 35,
      //     'state_price':35,
      //     'metro_price':40,
      //     'rest_price':43,
      //     'special_price':54
      //   },
      //   {
      //     'weight' : 1,
      //     'city_price': 62,
      //     'state_price':62,
      //     'metro_price':72,
      //     'rest_price':76,
      //     'special_price':98
      //   },
      //   {
      //     'weight' : 1.5,
      //     'city_price': 88,
      //     'state_price':88,
      //     'metro_price':105,
      //     'rest_price':109,
      //     'special_price':142
      //   },
      //   {
      //     'weight' : 2,
      //     'city_price': 115,
      //     'state_price':115,
      //     'metro_price':139,
      //     'rest_price':142,
      //     'special_price':186
      //   },
      //   {
      //     'weight' : 2.5,
      //     'city_price': 141,
      //     'state_price':141,
      //     'metro_price':172,
      //     'rest_price':175,
      //     'special_price':230
      //   },
      //   {
      //     'weight' : 3,
      //     'city_price': 168,
      //     'state_price':168,
      //     'metro_price':205,
      //     'rest_price':208,
      //     'special_price':274
      //   },
      //   {
      //     'weight' : 3.5,
      //     'city_price': 194,
      //     'state_price':194,
      //     'metro_price':238,
      //     'rest_price':241,
      //     'special_price':318
      //   },
      //   {
      //     'weight' : 4,
      //     'city_price': 220,
      //     'state_price':220,
      //     'metro_price':271,
      //     'rest_price':274,
      //     'special_price':362
      //   },
      //   {
      //     'weight' :4.5,
      //     'city_price': 246,
      //     'state_price':246,
      //     'metro_price':304,
      //     'rest_price':307,
      //     'special_price':406
      //   },
      //   {
      //     'weight' : 5,
      //     'city_price': 273,
      //     'state_price':273,
      //     'metro_price':337,
      //     'rest_price':340,
      //     'special_price':450
      //   },
      //   {
      //     'weight' : 5.5,
      //     'city_price': 299,
      //     'state_price':299,
      //     'metro_price':370,
      //     'rest_price':373,
      //     'special_price':494
      //   },
      //   {
      //     'weight' : 6,
      //     'city_price': 326,
      //     'state_price':326,
      //     'metro_price':403,
      //     'rest_price':406,
      //     'special_price':538
      //   },
      //   {
      //     'weight' :6.5,
      //     'city_price':352 ,
      //     'state_price':352,
      //     'metro_price':435,
      //     'rest_price':439,
      //     'special_price':582
      //   },
      //   {
      //     'weight' : 7,
      //     'city_price': 378,
      //     'state_price':378,
      //     'metro_price':469,
      //     'rest_price':472,
      //     'special_price':626
      //   },
      //   {
      //     'weight' : 7.5,
      //     'city_price': 405,
      //     'state_price':405,
      //     'metro_price':501,
      //     'rest_price':505,
      //     'special_price':670
      //   },
      //   {
      //     'weight' : 8,
      //     'city_price':432 ,
      //     'state_price':432,
      //     'metro_price':534,
      //     'rest_price':538,
      //     'special_price':714
      //   },
      //   {
      //     'weight' : 8.5,
      //     'city_price': 458,
      //     'state_price':458,
      //     'metro_price':568,
      //     'rest_price':571,
      //     'special_price':758
      //   },
      //   {
      //     'weight' : 9,
      //     'city_price': 484,
      //     'state_price':484,
      //     'metro_price':600,
      //     'rest_price':604,
      //     'special_price':802
      //   },
      //   {
      //     'weight' : 9.5,
      //     'city_price': 510,
      //     'state_price':510,
      //     'metro_price':633,
      //     'rest_price':637,
      //     'special_price':846
      //   },
      //   {
      //     'weight' : 10,
      //     'city_price': 537,
      //     'state_price':537,
      //     'metro_price':667,
      //     'rest_price':670,
      //     'special_price':890
      //   },
      //   {
      //     'weight' : 10.5,
      //     'city_price': 563,
      //     'state_price':563,
      //     'metro_price':700,
      //     'rest_price':703,
      //     'special_price':934
      //   },
      //   {
      //     'weight' : 11,
      //     'city_price':590,
      //     'state_price':590,
      //     'metro_price':733,
      //     'rest_price':736,
      //     'special_price':978
      //   },
      //   {
      //     'weight' : 11.5,
      //     'city_price': 616,
      //     'state_price':616,
      //     'metro_price':766,
      //     'rest_price':769,
      //     'special_price':1022
      //   },
      //   {
      //     'weight' : 12,
      //     'city_price': 642,
      //     'state_price':642,
      //     'metro_price':799,
      //     'rest_price':802,
      //     'special_price':1066
      //   },
      //   {
      //     'weight' : 12.5,
      //     'city_price': 669,
      //     'state_price':669,
      //     'metro_price':832,
      //     'rest_price':836,
      //     'special_price':1110
      //   },
      //   {
      //     'weight' : 13,
      //     'city_price': 695,
      //     'state_price':695,
      //     'metro_price':865,
      //     'rest_price':869,
      //     'special_price':1154
      //   },
      //   {
      //     'weight' : 13.5,
      //     'city_price': 722,
      //     'state_price':722,
      //     'metro_price':898,
      //     'rest_price':902,
      //     'special_price':1198
      //   },
      //   {
      //     'weight' : 14,
      //     'city_price': 748,
      //     'state_price':748,
      //     'metro_price':931,
      //     'rest_price':935,
      //     'special_price':1242
      //   },
      //   {
      //     'weight' : 14.5,
      //     'city_price': 774,
      //     'state_price':774,
      //     'metro_price':964,
      //     'rest_price':968,
      //     'special_price':1286
      //   },
      //   {
      //     'weight' : 15,
      //     'city_price': 801,
      //     'state_price':801,
      //     'metro_price':997,
      //     'rest_price':1001,
      //     'special_price':1374
      //   },
      //   {
      //     'weight' : 15.5,
      //     'city_price': 827,
      //     'state_price':827,
      //     'metro_price':1030,
      //     'rest_price':1034,
      //     'special_price':1374
      //   },
      //   {
      //     'weight' : 16,
      //     'city_price': 853,
      //     'state_price':853,
      //     'metro_price':1063,
      //     'rest_price':1067,
      //     'special_price':1418
      //   },
      //   {
      //     'weight' : 16.5,
      //     'city_price': 880,
      //     'state_price':880,
      //     'metro_price':1096,
      //     'rest_price':1100,
      //     'special_price':1462
      //   },
      //   {
      //     'weight' : 17,
      //     'city_price': 906,
      //     'state_price':906,
      //     'metro_price':1128,
      //     'rest_price':1132,
      //     'special_price':1506
      //   },
      //   {
      //     'weight' : 17.5,
      //     'city_price': 933,
      //     'state_price':933,
      //     'metro_price':1161,
      //     'rest_price':1165,
      //     'special_price':1550
      //   },
      //   {
      //     'weight' : 18,
      //     'city_price': 959,
      //     'state_price':959,
      //     'metro_price':1195,
      //     'rest_price':1199,
      //     'special_price':1594
      //   },
      //   {
      //     'weight' : 18.5,
      //     'city_price': 985,
      //     'state_price':985,
      //     'metro_price':1228,
      //     'rest_price':1132,
      //     'special_price':1638
      //   },
      //   {
      //     'weight' : 19,
      //     'city_price': 1012,
      //     'state_price':1012,
      //     'metro_price':1261,
      //     'rest_price':1265,
      //     'special_price':1682
      //   },
      //   {
      //     'weight' : 19.5,
      //     'city_price': 1038,
      //     'state_price':1038,
      //     'metro_price':1294,
      //     'rest_price':1298,
      //     'special_price':1726
      //   },
      //   {
      //     'weight' : 20,
      //     'city_price': 1065,
      //     'state_price':1065,
      //     'metro_price':1327,
      //     'rest_price':1331,
      //     'special_price':1770
      //   },
      //   {
      //     'weight' : 20.5,
      //     'city_price': 1091,
      //     'state_price':1091,
      //     'metro_price':1360,
      //     'rest_price':1364,
      //     'special_price':1814
      //   },
      //   {
      //     'weight' : 21,
      //     'city_price': 1118,
      //     'state_price':1118,
      //     'metro_price':1393,
      //     'rest_price':1397,
      //     'special_price':1858
      //   },
      //   {
      //     'weight' : 21.5,
      //     'city_price': 1144,
      //     'state_price':1144,
      //     'metro_price':1426,
      //     'rest_price':1430,
      //     'special_price':1902
      //   },
      //   {
      //     'weight' : 22,
      //     'city_price': 1170,
      //     'state_price':1170,
      //     'metro_price':1459,
      //     'rest_price':1463,
      //     'special_price':1946
      //   },
      //   {
      //     'weight' : 22.5,
      //     'city_price': 1197,
      //     'state_price':1197,
      //     'metro_price':1492,
      //     'rest_price':1496,
      //     'special_price':1990
      //   },
      //   {
      //     'weight' : 23,
      //     'city_price': 1223,
      //     'state_price':1223,
      //     'metro_price':1525,
      //     'rest_price':1529,
      //     'special_price':2034
      //   },
      //   {
      //     'weight' : 23.5,
      //     'city_price': 1250,
      //     'state_price':1250,
      //     'metro_price':1558,
      //     'rest_price':1562,
      //     'special_price':2078
      //   },
      //   {
      //     'weight' : 24,
      //     'city_price': 1276,
      //     'state_price':1276,
      //     'metro_price':1591,
      //     'rest_price':1595,
      //     'special_price':2122
      //   },
      //   {
      //     'weight' : 24.5,
      //     'city_price': 1302,
      //     'state_price':1302,
      //     'metro_price':1624,
      //     'rest_price':1628,
      //     'special_price':2166
      //   },
      //   {
      //     'weight' : 25,
      //     'city_price': 1359,
      //     'state_price':1329,
      //     'metro_price':1657,
      //     'rest_price':1661,
      //     'special_price':2210
      //   },
      //   {
      //     'weight' : 25.5,
      //     'city_price': 1355,
      //     'state_price':1355,
      //     'metro_price':11690,
      //     'rest_price':1694,
      //     'special_price':2254
      //   },
      //   {
      //     'weight' : 26,
      //     'city_price': 1381,
      //     'state_price':1381,
      //     'metro_price':1722,
      //     'rest_price':1726,
      //     'special_price':2298
      //   },
      //   {
      //     'weight' : 26.5,
      //     'city_price': 1408,
      //     'state_price':1408,
      //     'metro_price':1756,
      //     'rest_price':1760,
      //     'special_price':2342
      //   },
      //   {
      //     'weight' : 27,
      //     'city_price': 1434,
      //     'state_price':1434,
      //     'metro_price':1789,
      //     'rest_price':1793,
      //     'special_price':2386
      //   },
      //   {
      //     'weight' : 27.5,
      //     'city_price': 1460,
      //     'state_price':1460,
      //     'metro_price':1822,
      //     'rest_price':1826,
      //     'special_price':2430
      //   },
      //   {
      //     'weight' : 28,
      //     'city_price': 1487,
      //     'state_price':1487,
      //     'metro_price':1855,
      //     'rest_price':1859,
      //     'special_price':2474
      //   },
      //   {
      //     'weight' : 28.5,
      //     'city_price': 1513,
      //     'state_price':1513,
      //     'metro_price':1888,
      //     'rest_price':1892,
      //     'special_price':2518
      //   },
      //   {
      //     'weight' : 29,
      //     'city_price': 1540,
      //     'state_price':1540,
      //     'metro_price':1921,
      //     'rest_price':1924,
      //     'special_price':2562
      //   },
      //   {
      //     'weight' : 29.5,
      //     'city_price': 1566,
      //     'state_price':1566,
      //     'metro_price':1953,
      //     'rest_price':1957,
      //     'special_price':2606
      //   },
      //   {
      //     'weight' : 30,
      //     'city_price': 1592,
      //     'state_price':1592,
      //     'metro_price':1986,
      //     'rest_price':1990,
      //     'special_price':2650
      //   },
      // ],
      // options: [
      //   {value:'1', text: 'Apparel (upto INR 300)', marketfee_percent:'13', min_val: 0, max_val: 300},
      //   {value:'2', text: 'Apparel (more than INR 300)', marketfee_percent:'17', min_val: 300, max_val: 'false'},
      //   {value:'3', text: 'Apparel - Sarees and Dress Materials (upto INR 500)', marketfee_percent:'13.5', min_val:'false', max_val: 500},
      //   {value:'4', text: 'Apparel - Sarees and Dress Materials (INR 501 to INR 1000)' , marketfee_percent:'17', min_val: 500, max_val: 1000},
      //   {value:'5', text: 'Apparel - Sarees and Dress Materials (more than INR 1000)' , marketfee_percent:'19', min_val: 1000, max_val: 'false'},
      //   {value:'6', text: 'apparel - Accessories',marketfee_percent:'17', min_val:'false', max_val: 'false'},
      //   {value:'7', text: 'apparel - Innerwear',marketfee_percent:'13', min_val:'false', max_val: 'false'},
      //   {value:'8', text: 'apparel - Sleepwear',marketfee_percent:'13', min_val:'false', max_val: 'false'},
      //   {value:'9', text: 'apparel - Apparel - Men T-shirts (except Polos, Tank tops and full sleeve tops)',marketfee_percent:'15.50', min_val:'false', max_val: 'false'},
      //   {value:'10', text: 'apparel - Fashion Jewellery',marketfee_percent:'22.50', min_val:'false', max_val: 'false'},
      //   {value:'11', text: 'Fine Jewellery - Gold Coins',marketfee_percent:'2.50', min_val:'false', max_val: 'false'},
      //   {value:'12', text: 'Fine Jewellery - Studded',marketfee_percent:'8', min_val:'false', max_val: 'false'},
      //   {value:'13', text: 'Fine Jewellery - Unstudded and Solitaire',marketfee_percent:'5', min_val:'false', max_val: 'false'},
      //   {value:'14', text: 'Silver Jewellery',marketfee_percent:'8', min_val:'false', max_val: 'false'},
      //   {value:'15', text: 'Handbags',marketfee_percent:'11', min_val:'false', max_val: 'false'},
      //   {value:'16', text: 'Backpacks',marketfee_percent:'10', min_val:'false', max_val: 'false'},
      //   {value:'17', text: 'Luggage - Suitcase & Trolleys',marketfee_percent:'6', min_val:'false', max_val: 'false'},
      //   {value:'18', text: 'Luggage - Travel Accessories',marketfee_percent:'11', min_val:'false', max_val: 'false'},
      //   {value:'19', text: 'Luggage - Other Subcategorie',marketfee_percent:'5.50', min_val:'false', max_val: 'false'},
      //   {value:'20', text: 'Shoes',marketfee_percent:'14', min_val:'false', max_val: 'false'},
      //   {value:'21', text: 'Flip Flop & Slippers',marketfee_percent:'10.50', min_val:'false', max_val: 'false'},
      //   {value:'22', text: 'Shoes - Sandals & floaters',marketfee_percent:'10.50', min_val:'false', max_val: 'false'},
      //   {value:'23', text: 'Shoes - Kids Footwear',marketfee_percent:'10.50', min_val:'false', max_val: 'false'},
      //   {value:'24', text: 'Eyewear',marketfee_percent:'10', min_val:'false', max_val: 'false'},
      //   {value:'25', text: 'Wallets',marketfee_percent:'11.50', min_val:'false', max_val: 'false'},
      //   {value:'26', text: 'Watches',marketfee_percent:'13.50', min_val:'false', max_val: 'false'},
      //   {value:'27', text: 'Cables-Electronics, PC, Wireless',marketfee_percent:'20', min_val:'false', max_val: 'false'},
      //   {value:'28', text:'Camera Accessories',marketfee_percent:'11', min_val:'false', max_val: 'false' },
      //   {value:'29', text: 'Camera Lenses' ,marketfee_percent:'7', min_val:'false', max_val: 'false'},
      //   {value:'30', text: 'Camera and Camcorder' ,marketfee_percent:'5', min_val:'false', max_val: 'false'},
      //   {value:'31', text:'Cases, Covers, Skins, Screen Gaurds' ,marketfee_percent:'3', min_val:'false', max_val: 150},
      //   {value:'32', text:'Cases, Covers, Skins, Screen Gaurds' ,marketfee_percent:'18', min_val: 150, max_val: 300},
      //   {value:'33', text:'Cases, Covers, Skins, Screen Gaurds' ,marketfee_percent:'20', min_val: 300, max_val: 500},
      //   {value:'34', text:'Cases, Covers, Skins, Screen Gaurds' ,marketfee_percent:'25', min_val:500, max_val:'false'},
      //   {value:'35', text: 'Desktops',marketfee_percent:'6.50', min_val:'false', max_val: 'false'},
      //   {value:'36', text: 'Electronic Accessories (Electronics, PC, & Wireless)',marketfee_percent:'17', min_val:'false', max_val: 'false'},
      //   {value:'37', text: 'Electronic Devices (except TV, Camera & Camcorder, Camera Lenses and Accessories, GPS Devices, Speakers)',marketfee_percent:'8.50', min_val:'false', max_val: 'false'},
      //   {value:'38', text: 'Entertainment Collectibles' ,marketfee_percent:'13', min_val:'false', max_val:300},
      //   {value:'39', text: 'Entertainment Collectibles' ,marketfee_percent:'17', min_val:300, max_val:'false'},
      //   {value:'40', text:'Smart Watches and Accessories',marketfee_percent:'14.50', min_val:'false', max_val: 'false' },
      //   {value:'41', text:'GPS Devices',marketfee_percent:'13.50', min_val:'false', max_val: 'false' },
      //   {value:'42', text:'Hard Disks',marketfee_percent:'8.50', min_val:'false', max_val: 'false' },
      //   {value:'43', text:'Headsets, Headphones and Earphones',marketfee_percent:'15', min_val:'false', max_val: 'false' },
      //   {value:'44', text:'Keyboards and Mouse',marketfee_percent:'13', min_val:'false', max_val: 'false' },
      //   {value:'45', text:'Kindle Accessories',marketfee_percent:'25', min_val:'false', max_val: 'false' },
      //   {value:'46', text:'Laptop Bags and Sleeves',marketfee_percent:'10', min_val:'false', max_val: 'false' },
      //   {value:'47', text:'Laptop Battery',marketfee_percent:'14', min_val:'false', max_val: 'false' },
      //   {value:'48', text:'Laptops',marketfee_percent:'5', min_val:'false', max_val: 'false' },
      //   {value:'49', text:'Memory Cards',marketfee_percent:'12', min_val:'false', max_val: 'false' },
      //   {value:'50', text:'Mobile Phones & Tablets (including Graphic Tablets)',marketfee_percent:'5', min_val:'false', max_val: 'false' },
      //   {value:'51', text:'Modems and Networking Devices',marketfee_percent:'14', min_val:'false', max_val: 'false' },
      //   {value:'52', text:'Monitors',marketfee_percent:'6.50', min_val:'false', max_val: 'false' },
      //   {value:'53', text:'PC Components (RAM, Motherboards)',marketfee_percent:'7', min_val:'false', max_val: 'false' },
      //   {value:'54', text:'Power Banks',marketfee_percent:'18', min_val:'false', max_val: 'false' },
      //   {value:'55', text:'Printers & Scanners',marketfee_percent:'8', min_val:'false', max_val: 'false' },
      //   {value:'56', text:'Software',marketfee_percent:'11.50', min_val:'false', max_val: 'false' },
      //   {value:'57', text:'Speakers',marketfee_percent:'11', min_val:'false', max_val: 'false' },
      //   {value:'58', text:'Television',marketfee_percent:'6', min_val:'false', max_val: 'false' },
      //   {value:'59', text:'Landline Phones',marketfee_percent:'5', min_val:'false', max_val: 'false' },
      //   {value:'60', text:'USB Flash Drives (Pen Drives)',marketfee_percent:'16', min_val:'false', max_val: 'false' },
      //   {value:'61', text:'Grocery & Gormet Food',marketfee_percent:'4.50', min_val:'false', max_val: 'false'  },
      //   {value:'62', text:'Pet Products',marketfee_percent:'6.5', min_val:'false', max_val:250 },
      //   {value:'63', text:'Pet Products',marketfee_percent:'11', min_val:250, max_val:'false' },
      //   {value:'64', text:'Beauty - Fragnance/Perfumes',marketfee_percent:'7.5', min_val:'false', max_val:250 },
      //   {value:'65', text:'Beauty - Fragnance/Perfumes',marketfee_percent:'12.5', min_val:250, max_val:'false' },
      //   {value:'66', text:'Beauty Products',marketfee_percent:'4.50', min_val:'false', max_val: 'false'  },
      //   {value:'67', text:'Prescription Medicine',marketfee_percent:'4.50', min_val:'false', max_val: 'false'  },
      //   {value:'68', text:'Health & Personal Care(HPC) - Medical Equipment & Contact Lens',marketfee_percent:'8', min_val:'false', max_val: 'false'  },
      //   {value:'69', text:'Health & Personal Care(HPC) - Household Supplies, Personal Care & Ayurveda',marketfee_percent:'4.50', min_val:'false', max_val: 'false'  },
      //   {value:'70', text:'Health & Personal Care(HPC) - Nutrition',marketfee_percent:'9', min_val:'false', max_val: 'false'  },
      //   {value:'71', text:'Health & Personal Care(HPC) - Other Subcategories',marketfee_percent:'11', min_val:'false', max_val: 'false'  },
      //   {value:'72', text:'Luxury Beauty',marketfee_percent:'4.50', min_val:'false', max_val: 'false'  },
      //   {value:'73', text:'Car Cradles, Lens Kits and Tablet Cases',marketfee_percent:'21', min_val:'false', max_val: 'false'  },
      //   {value:'74', text:'Personal Care Appliances - Electric Massagers',marketfee_percent:'9', min_val:'false', max_val: 'false'  },
      //   {value:'75', text:'Personal Care Appliances(Grooming & Styling)',marketfee_percent:'9.50', min_val:'false', max_val: 'false'  },
      //   {value:'76', text:'Personal Care Appliances(excl. Grooming and Styling)',marketfee_percent:'7.50', min_val:'false', max_val: 'false'  },
      //   {value:'77', text:'Bean Bags & Inflatables',marketfee_percent:'10', min_val:'false', max_val: 'false'},
      //   {value:'78', text:'Clocks',marketfee_percent:'8', min_val:'false', max_val: 'false'},
      //   {value:'79', text:'Furniture',marketfee_percent:'14', min_val:'false', max_val: 15000},
      //   {value:'80', text:'Furniture',marketfee_percent:'10', min_val:15000, max_val: 'false'},
      //   {value:'81', text:'Home - Fragnance & Candles',marketfee_percent:'10.50', min_val:'false', max_val: 'false'},
      //   {value:'82', text:'Home - Bedsheets, Blankets & Covers',marketfee_percent:'9', min_val:'false', max_val: 'false'},
      //   {value:'83', text:'Home Furnishing',marketfee_percent:'12', min_val:'false', max_val: 'false'},
      //   {value:'84', text:'Home Improvement - Accessories',marketfee_percent:'6.5', min_val:'false', max_val: 250},
      //   {value:'85', text:'Home Improvement - Accessories',marketfee_percent:'11.5', min_val:250, max_val: 'false'},
      //   {value:'86', text:'Home Improvement (excl. Accessories)',marketfee_percent:'8', min_val:'false', max_val: 'false'},
      //   {value:'87', text:'Home Improvement - Ladders, Kitchen, Bath Fixtures & Home Security Systems',marketfee_percent:'8', min_val:'false', max_val: 'false'},
      //   {value:'88', text:'Home Storage',marketfee_percent:'13', min_val:'false', max_val: 'false'},
      //   {value:'89', text:'Home - Other Subcategories',marketfee_percent:'17', min_val:'false', max_val: 'false'},
      //   {value:'90', text:'Home - waste & Recycling',marketfee_percent:'6', min_val:'false', max_val: 'false'},
      //   {value:'91', text:'Indoor Lighting (excluding LED bulbs and Battens)',marketfee_percent:'16', min_val:'false', max_val: 'false'},
      //   {value:'92', text:'Indoor Lighting – Wall, ceiling fixture lights, lamp bases, lamp shades and Smart Lighting',marketfee_percent:'13.50', min_val:'false', max_val: 'false'},
      //   {value:'93', text:'LED bulbs and Battens',marketfee_percent:'7', min_val:'false', max_val: 'false'},
      //   {value:'94', text:'Lawn and Garden - Chemical Pest Control',marketfee_percent:'5', min_val:'false', max_val: 'false'},
      //   {value:'95', text:'Lawn and Garden - Solar Devices',marketfee_percent:'5', min_val:'false', max_val: 'false'},
      //   {value:'96', text:'Lawn and Garden - Others',marketfee_percent:'10', min_val:'false', max_val: 15000},
      //   {value:'97', text:'Lawn and Garden - Others',marketfee_percent:'5', min_val:15000, max_val: 'false'},
      //   {value:'98', text:'Lawn and Garden - Plants, Seeds & Bulbs',marketfee_percent:'7', min_val:'false', max_val:300},
      //   {value:'99', text:'Lawn and Garden - Plants, Seeds & Bulbs',marketfee_percent:'10', min_val:300, max_val: 'false'},
      //   {value:'100', text:'Outdoor Equipments (Pumps, Generators, Grilling, Agricultural Machinery, Saws, Tillers, Pressure Washers, Lawn Mowers & String Trimmers)',marketfee_percent:'5.50', min_val:'false', max_val: 'false'},
      //   {value:'101', text:'Kitchen - Non Appliances',marketfee_percent:'5', min_val:'false', max_val: 300},
      //   {value:'102', text:'Kitchen - Non Appliances',marketfee_percent:'11.5', min_val:300, max_val:'false'},
      //   {value:'103', text:'Kitchen - Glassware & Ceramicwave',marketfee_percent:'11', min_val:'false', max_val:'false'},
      //   {value:'104', text:'Kitchen - Gas Stoves & Pressure Cookers',marketfee_percent:'7', min_val:'false', max_val:'false'},
      //   {value:'105', text:'Large Appliances (excl. Accessories and Chimneys)',marketfee_percent:'5', min_val:'false', max_val:'false'},
      //   {value:'106', text:'Large Appliances - Accessories',marketfee_percent:'13', min_val:'false', max_val:'false'},
      //   {value:'107', text:'Large Appliances - Chimneys',marketfee_percent:'7', min_val:'false', max_val:'false'},
      //   {value:'108', text:'Large Appliances - Refrigerators',marketfee_percent:'4.50', min_val:'false', max_val:'false'},
      //   {value:'109', text:'Small Appliances',marketfee_percent:'5.5', min_val:'false', max_val:5000},
      //   {value:'110', text:'Small Appliances',marketfee_percent:'6.5', min_val:5000, max_val:'false'},
      //   {value:'111', text:'Bicycles',marketfee_percent:'8', min_val:'false', max_val:'false'},
      //   {value:'112', text:'Gym Equipments',marketfee_percent:'9', min_val:'false', max_val:'false'},
      //   {value:'113', text:'Sports - Cricket & Badminton Equipments',marketfee_percent:'6', min_val:'false', max_val:'false'},
      //   {value:'114', text:'Sports - Football, Volleyball, Basketball, Throwball',marketfee_percent:'6', min_val:'false', max_val:'false'},
      //   {value:'115', text:'Sports - Tennis, Table Tennis & Squash',marketfee_percent:'6', min_val:'false', max_val:'false'},
      //   {value:'116', text:'Sports - swimming',marketfee_percent:'6', min_val:'false', max_val:'false'},
      //   {value:'117', text:'Sports - Others',marketfee_percent:'9', min_val:'false', max_val:250},
      //   {value:'118', text:'Sports - Others',marketfee_percent:'11.5', min_val:250, max_val:'false'},
      //   {value:'119', text:'Business, Industrial & Scientific (BISS) Equipment - Lab Equipment & Occupational Safety Supplies (incl. Face Masks & Respirators)',marketfee_percent:'11.5', min_val:'false', max_val:150000},
      //   {value:'120', text:'Business, Industrial & Scientific (BISS) Equipment - Lab Equipment & Occupational Safety Supplies (incl. Face Masks & Respirators)',marketfee_percent:'5', min_val:150000, max_val:'false'},
      //   {value:'121', text:'Business, Industrial & Scientific (BISS) Equipment - Food equipment, Material Handling, Professional cleaning, Medical and Healthcare Supplies',marketfee_percent:'5', min_val:'false', max_val:'false'},
      //   {value:'122', text:'Business, Industrial & Scientific (BISS) Small Instruments - Power and Hand tools, Measuring instruments, Tapes and Adhesives',marketfee_percent:'8', min_val:'false', max_val:150000},
      //   {value:'123', text:'Business, Industrial & Scientific (BISS) Small Instruments - Power and Hand tools, Measuring instruments, Tapes and Adhesives',marketfee_percent:'5', min_val:15000, max_val:'false'},
      //   {value:'124', text:'Masks, Gloves, Personal Protective Equipment & Medical Apparel',marketfee_percent:'5', min_val:'false', max_val:'false'},
      //   {value:'125', text:'Industrial Supplies',marketfee_percent:'8', min_val:'false', max_val:15000},
      //   {value:'126', text:'Industrial Supplies',marketfee_percent:'5', min_val:15000, max_val:'false'},
      //   {value:'127', text:'Office Products',marketfee_percent:'8', min_val:'false', max_val:'false'},
      //   {value:'128', text:'Office Electronics Products',marketfee_percent:'9', min_val:'false', max_val:'false'},
      //   {value:'129', text:'Books',marketfee_percent:'2', min_val:'false', max_val:250},
      //   {value:'130', text:'Books',marketfee_percent:'4', min_val:250, max_val:500},
      //   {value:'131', text:'Books',marketfee_percent:'9', min_val:500, max_val:1000},
      //   {value:'132', text:'Books',marketfee_percent:'13', min_val:1300, max_val:'false'},
      //   {value:'133', text:'Movies',marketfee_percent:'6.50', min_val:'false', max_val:'false'},
      //   {value:'134', text:'Music',marketfee_percent:'6.50', min_val:'false', max_val:'false'},
      //   {value:'135', text:'Musical Instruments( excluding Guitars & Keyboards) ',marketfee_percent:'7.50', min_val:'false', max_val:'false'},
      //   {value:'136', text:'Musical Instruments - Guitars',marketfee_percent:'7.50', min_val:'false', max_val:'false'},
      //   {value:'137', text:'Musical Instruments - Keyboards',marketfee_percent:'5', min_val:'false', max_val:'false'},
      //   {value:'138', text:'Musical Instruments - DJ & VJ Equipment',marketfee_percent:'8', min_val:'false', max_val:'false'},
      //   {value:'139', text:'Recording & Computer',marketfee_percent:'8', min_val:'false', max_val:'false'},
      //   {value:'140', text:'Cables & Leads',marketfee_percent:'8', min_val:'false', max_val:'false'},
      //   {value:'141', text:'Microphones',marketfee_percent:'8', min_val:'false', max_val:'false'},
      //   {value:'142', text:'PA & Stage',marketfee_percent:'8', min_val:'false', max_val:'false'},
      //   {value:'143', text:'Video Games - Online game services',marketfee_percent:'2', min_val:'false', max_val:'false'},
      //   {value:'144', text:'Video Games - Accessories',marketfee_percent:'9', min_val:'false', max_val:500},
      //   {value:'145', text:'Video Games - Accessories',marketfee_percent:'12', min_val:500, max_val:'false'},
      //   {value:'146', text:'Video Games - Consoles',marketfee_percent:'7', min_val:500, max_val:'false'},
      //   {value:'147', text:'Video Games',marketfee_percent:'7', min_val:'false', max_val:'false'},
      //   {value:'148', text:'Baby Products',marketfee_percent:'6', min_val:'false', max_val:'false'},
      //   {value:'149', text:'Baby Hardlines - Swings, Bouncers and Rockers, Carriers, Walkers',marketfee_percent:'8', min_val:'false', max_val:'false'},
      //   {value:'150', text:'Baby Safety - Gaurds & Locks',marketfee_percent:'8', min_val:'false', max_val:'false'},
      //   {value:'151', text:'Baby Room Decor',marketfee_percent:'8', min_val:'false', max_val:'false'},
      //   {value:'152', text:'Baby Furniture',marketfee_percent:'8', min_val:'false', max_val:'false'},
      //   {value:'153', text:'Baby Car Seats & Accessories',marketfee_percent:'8', min_val:'false', max_val:'false'},
      //   {value:'154', text:'Baby Strollers, Buggies & Prams',marketfee_percent:'8', min_val:'false', max_val:'false'},
      //   {value:'155', text:'Craft Materials',marketfee_percent:'8', min_val:'false', max_val:'false'},
      //   {value:'156', text:'Toys',marketfee_percent:'9.50', min_val:'false', max_val:'false'},
      //   {value:'157', text:'Toys - Drones',marketfee_percent:'10.50', min_val:'false', max_val:'false'},
      //   {value:'158', text:'Automotive - Lubricants, Car Parts and Helmets, Vehicle Care and Tools',marketfee_percent:'6.50', min_val:'false', max_val:'false'},
      //   {value:'159', text:'Automotive - Tyres and Rims',marketfee_percent:'5', min_val:'false', max_val:'false'},
      //   {value:'160', text:'Automotive - Vehicles',marketfee_percent:'2', min_val:'false', max_val:'false'},
      //   {value:'161', text:'Automotive - Other Subcategories',marketfee_percent:'20', min_val:'false', max_val:'false'},
      //   {value:'162', text:'Automotive Accessories (Floor Mats, Seat/Car/Bike Covers) and Riding Gear (Face Covers and Gloves)',marketfee_percent:'13', min_val:'false', max_val:'false'},
      //   {value:'163', text:'Car Electronics Devices',marketfee_percent:'5.50', min_val:'false', max_val:'false'},
      //   {value:'164', text:'Coin Collectibles',marketfee_percent:'15', min_val:'false', max_val:'false'},
      //   {value:'165', text:'Consumable Gift Cards',marketfee_percent:'5', min_val:'false', max_val:'false'},
      //   {value:'166', text:'Fine Art',marketfee_percent:'20', min_val:'false', max_val:'false'},
      //   {value:'167', text:'Silver Coins & Bars',marketfee_percent:'2.50', min_val:'false', max_val:'false'},
      //   {value:'168', text:'Sports Collectibles',marketfee_percent:'13', min_val:'false', max_val:300},
      //   {value:'169', text:'Sports Collectibles',marketfee_percent:'17', min_val:300, max_val:'false'},
      //   {value:'170', text:'Wall Art (Paintings, Posters, Decorative Stickers and Art Prints)',marketfee_percent:'13.50', min_val:'false', max_val:'false'},
      //   {value:'171', text:'Warranty Services',marketfee_percent:'38', min_val:'false', max_val:'false'},
      // ],
      // options:[],
      options: [],
      arr: [],
      shipping_table1:[],
      shipping_table2:[],
      shipping_table3:[],
      shipping_table4:[],
      shipping_table5:[],
      item: {
        text:'',
        marketfee_percent:'',
        min_val:'',
        max_val:'',
      },
      modal:{
        text : '',
        marketfee_percent : '',
        data : '',
        category:'',
        charges:''
      },
      // PartOne:[],
      selling_price: "",
      closing_fee: "",
      closing_fees:[],
      errortext: "",
      show_shipping: true,
      show_with_shipping: false,
      show_fee:false,
      show_total:false,
      user_loggedin:false,
      show_product_category:false,
      weight:"",
      shipping_category:"",
      shipping_charges: "",
      arr_length:''
    }
  },
  computed: {
    total(){
      return parseFloat(parseFloat(this.market_fee) + parseFloat(this.closing_fees.charges) + parseFloat(this.shipping_charges)).toFixed(2)
    },
    gst_total(){
      return parseFloat(this.total*0.18).toFixed(2)
    },
    grand_total(){
      return parseFloat(parseFloat(this.total) + parseFloat(this.gst_total)).toFixed(2) 
    },
    profit(){
      return parseFloat(parseFloat(this.selling_price) - parseFloat(this.grand_total)).toFixed(2)
    },
    market_fee(){
      console.log(this.item.marketfee_percent/100)
      console.log(this.selling_price)
      return (this.item.marketfee_percent/100) * this.selling_price
    },
  },
  async created(){
    db.collection("options").get().then((querySnapshot) => {
        querySnapshot.forEach((doc) => {
          let tempdata = {}
          tempdata.value = doc.data().value
          tempdata.text = doc.data().text
          tempdata.marketfee_percent = doc.data().marketfee_percent
          tempdata.min_val = doc.data().min_val
          tempdata.max_val = doc.data().max_val
          tempdata.doc_id = doc.id
          this.options.push(tempdata)
          let tempoptions = this.options;
          tempoptions = tempoptions.sort((a,b) => {
            return a.value - b.value
          })
          this.options = tempoptions
        });
      });
    db.collection("closing_fee").get().then((querySnapshot) => {
        querySnapshot.forEach((doc) => {
          let temp_closingfee = {}
          temp_closingfee = doc.data()
          temp_closingfee.category = doc.data().category
          temp_closingfee.charges = doc.data().charges
          temp_closingfee.doc_id = doc.id
          this.closing_fees.push(temp_closingfee)
          let tempclosingfee = this.closing_fees;
          tempclosingfee = tempclosingfee.sort((a,b) => {
            return a.charges - b.charges
          })
          this.closing_fees = tempclosingfee
        });
    });
    
    db.collection("shipping_fee").get().then((querySnapshot) => {
        querySnapshot.forEach((doc) => {
          let temp_shippingfee = {}
          temp_shippingfee = doc.data()
          // console.log(temp_shippingfee.weight)
          temp_shippingfee.doc_id = doc.id
          this.arr.push(temp_shippingfee)
          let temp = this.arr
          temp = temp.sort((a,b) => {
            return a.weight - b.weight
          })
          this.arr = temp
        });
    });

    db.collection("public_shippingfee").doc('JKFWUeAhMaMe7Sz29Jrq').get().then((doc) => {
      let temp_public_shippingfee = {}
      temp_public_shippingfee = doc.data()
      temp_public_shippingfee.doc_id = doc.id
      this.shipping_table1 = (temp_public_shippingfee)
      console.log(this.shipping_table1)
    });

    db.collection("public_shippingfee").doc('sPZctrs9YL1ayupWJiqM').get().then((doc) => {
      let temp_public_shippingfee = {}
      temp_public_shippingfee = doc.data()
      temp_public_shippingfee.doc_id = doc.id
      this.shipping_table2 = (temp_public_shippingfee)
      console.log(this.shipping_table2)
    });

    db.collection("public_shippingfee").doc('bAe1mjzLRR6eQMMfOXET').get().then((doc) => {
      let temp_public_shippingfee = {}
      temp_public_shippingfee = doc.data()
      temp_public_shippingfee.doc_id = doc.id
      this.shipping_table3 = (temp_public_shippingfee)
      console.log(this.shipping_table3)
    });
    
    db.collection("public_shippingfee").doc('NgzIQoUXhHMwxx5JeG1r').get().then((doc) => {
      let temp_public_shippingfee = {}
      temp_public_shippingfee = doc.data()
      temp_public_shippingfee.doc_id = doc.id
      this.shipping_table4 = (temp_public_shippingfee)
      console.log(this.shipping_table3)
    });

    db.collection("public_shippingfee").doc('M9R3ljeITyDzwnxGv6qe').get().then((doc) => {
      let temp_public_shippingfee = {}
      temp_public_shippingfee = doc.data()
      temp_public_shippingfee.doc_id = doc.id
      this.shipping_table5 = (temp_public_shippingfee)
      console.log(this.shipping_table3)
    });


    // user_status(){
    const user = auth.currentUser;
    if (user) {
      this.user_loggedin = true
      console.log(this.user_loggedin)
    } 
    else {
      this.user_loggedin = false
      console.log(this.user_loggedin)

      // No user is signed in.
      } 
    // }

  },
  methods: {  
    show(){
      this.showshippingtable = !this.showshippingtable
    },
    logout(){
      this.$store.dispatch('logout')
    },

    edit_shippingtable5_PartOne(data, doc_id, index){
      var modal = document.getElementById("shippingtable5_modal1");
      modal.style.display = "block"; 
      this.modal = data
      this.modal.doc_id = doc_id
      this.modal.index = index
      console.log(this.modal)
    },

    save_shippingtable5_PartOne(){
      let data = this.modal
      console.log(data)
      console.log(db.collection("public_shippingfee"))
      let temp_id = "PartOne." + data.index 
      db.collection("public_shippingfee").doc(data.doc_id).update({
        [`${temp_id}`] : {
          Region : this.modal.Region,
          north : this.modal.north,
          west : this.modal.west,
          south : this.modal.south,
          central : this.modal.central,
          east : this.modal.east,
          jandk : this.modal.jandk,
          kerala : this.modal.kerala,
          guwahati : this.modal.guwahati,
          rest : this.modal.rest,
        }
      })
      this.close_shippingtable5_PartOne()
    },

    close_shippingtable5_PartOne(){
      this.modal = null
      var modal = document.getElementById("shippingtable5_modal1");
      modal.style.display = "none";
    },

    edit_shippingtable3_PartOne(data, doc_id, index){
      var modal = document.getElementById("shippingtable3_modal1");
      modal.style.display = "block"; 
      this.modal = data
      this.modal.doc_id = doc_id
      this.modal.index = index
      console.log(this.modal)
    },

    save_shippingtable3_PartOne(){
      let data = this.modal
      console.log(data)
      console.log(db.collection("public_shippingfee"))
      let temp_id = "PartOne." + data.index 
      db.collection("public_shippingfee").doc(data.doc_id).update({
        [`${temp_id}`] : {
          Region : this.modal.Region,
          base_cost : this.modal.base_cost,
          excess_cost : this.modal.excess_cost,
          base_weight : this.modal.base_weight,
          excess_weight : this.modal.excess_weight,
        }
      })
      this.close_shippingtable3_PartOne()
    },

    close_shippingtable3_PartOne(){
      this.modal = null
      var modal = document.getElementById("shippingtable3_modal1");
      modal.style.display = "none";
    },

    edit_shippingtable2_PartOne(data, doc_id, index){
      var modal = document.getElementById("shippingtable2_modal1");
      modal.style.display = "block"; 
      this.modal = data
      this.modal.doc_id = doc_id
      this.modal.index = index
      console.log(this.modal)
    },

    save_shippingtable2_PartOne(){
      let data = this.modal
      console.log(data)
      console.log(db.collection("public_shippingfee"))
      let temp_id = "PartOne." + data.index 
      db.collection("public_shippingfee").doc(data.doc_id).update({
        [`${temp_id}`] : {
          Zone : this.modal.Zone,
          Region : this.modal.Region,
          first500 : this.modal.first500,
          excess500 : this.modal.excess500,
        }
      })
      this.close_shippingtable2_PartOne()
    },

    close_shippingtable2_PartOne(){
      this.modal = null
      var modal = document.getElementById("shippingtable2_modal1");
      modal.style.display = "none";
    },

    edit_shippingtable1_PartTwo(data, doc_id, index){
      var modal = document.getElementById("shippingtable1_modal2");
      modal.style.display = "block"; 
      this.modal = data
      this.modal.doc_id = doc_id
      this.modal.index = index
      console.log(this.modal)
    },

    save_shippingtable1_PartTwo(){
      let data = this.modal
      console.log(data)
      console.log(db.collection("public_shippingfee"))
      let temp_id = "PartTwo." + data.index 
      db.collection("public_shippingfee").doc(data.doc_id).update({
        [`${temp_id}`] : {
          head : this.modal.head,
          value : this.modal.value
        }
      })
      this.close_shippingtable1_PartTwo()
    },

    close_shippingtable1_PartTwo(){
      this.modal = null
      var modal = document.getElementById("shippingtable1_modal2");
      modal.style.display = "none";
    },


    edit_shippingtable1_PartOne(data, doc_id, index){
      var modal = document.getElementById("shippingtable1_modal1");
      modal.style.display = "block"; 
      this.modal = data
      this.modal.doc_id = doc_id
      this.modal.index = index
      console.log(this.modal)
    },

    save_shippingtable1_PartOne(){
      let data = this.modal
      console.log(data)
      console.log(db.collection("public_shippingfee"))
      let temp_id = "PartOne." + data.index 
      db.collection("public_shippingfee").doc(data.doc_id).update({
        [`${temp_id}`] : {
          Zone : this.modal.Zone,
          Region : this.modal.Region,
          air_first500 : this.modal.air_first500,
          air_excess500 : this.modal.air_excess500,
          ExpressTAT : this.modal.ExpressTAT,
          surface_first500 : this.modal.surface_first500,
          surface_excess500 : this.modal.surface_excess500,
          EconomyTAT : this.modal.EconomyTAT
        }
      })
      this.close_shippingtable1_PartOne()
    },

    close_shippingtable1_PartOne(){
      this.modal = null
      var modal = document.getElementById("shippingtable1_modal1");
      modal.style.display = "none";
    },

    edit_shippingfee(data){
      var modal = document.getElementById("shippingfee_modal");
      modal.style.display = "block"; 
      this.modal = data
      console.log(this.modal)
    },

    save_shippingfee(){
      let data = this.modal
      db.collection("shipping_fee").doc(data.doc_id).set({
        weight : this.modal.weight,
        city_price : this.modal.city_price,
        state_price : this.modal.state_price,
        rest_price : this.modal.rest_price,
        metro_price : this.modal.metro_price,
        special_price : this.modal.special_price
      })
      let temp = this.arr.find( ({ weight }) => weight ===  data.weight);
      console.log(temp)
      this.close_shippingfee_modal()
    },

    close_shippingfee_modal(){
      this.modal = null
      var modal = document.getElementById("shippingfee_modal");
      modal.style.display = "none";
    },

    edit_closingfee(data){
      var modal = document.getElementById("closingfee_modal");
      modal.style.display = "block"; 
      this.modal.data = data
      this.modal.category = data.category
      this.modal.charges = data.charges
    },

    save_closingfee_edit(){
      let data = this.modal.data
      db.collection("closing_fee").doc(data.doc_id).set({
        category : this.modal.category,
        charges : this.modal.charges,
      })
      let temp = this.closing_fees.find( ({ category }) => category ===  data.category);
      temp.category = this.modal.category
      temp.charges = this.modal.charges
      this.close_closingfee_modal()
    },

    close_closingfee_modal(){
      this.modal.category =null
      this.modal.charges = null
      var modal = document.getElementById("closingfee_modal");
      modal.style.display = "none";
    },

    edit_field(data){
      this.modal.data = data
      console.log(data)
      this.modal.text = data.text
      this.modal.marketfee_percent = data.marketfee_percent
      var modal = document.getElementById("myModal");
      modal.style.display = "block"; 
    },

    save_edit(){
      console.log(this.modal.data)
      let data = this.modal.data
      db.collection("options").doc(data.doc_id).set({
        value : data.value,
        text : this.modal.text,
        marketfee_percent : this.modal.marketfee_percent,
        min_val : data.min_val,
        max_val : data.max_val
      })
      let temp = this.options.find( ({ value }) => value ===  data.value);
      console.log(temp) 
      temp.text = this.modal.text
      temp.marketfee_percent = this.modal.marketfee_percent
      console.log(temp)
      this.close_modal()
    },

    close_modal(){
      this.modal.text =null
      this.modal.marketfee_percent = null
      this.modal.data = null
      var modal = document.getElementById("myModal");
      modal.style.display = "none";
      // console.log(this.modal)
    },
    
    add(){
      var optionsRef = db.collection("options");
      optionsRef.orderBy("value");
      console.log(optionsRef)
      // for(let i = 0; i<this.options.length; i++){
      //   console.log(this.options[i])
      //   db.collection('options').add({
      //     value: this.options[i].value,
      //     text: this.options[i].text,
      //     marketfee_percent: this.options[i].marketfee_percent,
      //     min_val: this.options[i].min_val,
      //     max_val: this.options[i].max_val 
      //   })
      //   .then((docRef) => {
      //       console.log("Document written with ID: ", docRef.id);
      //   })
      //   .catch((error) => {
      //       console.error("Error adding document: ", error);
      //   });
      // }

    },
    //   db.collection("options").orderBy("value", "asc")
    // },

    back(){
      this.show_shipping = true
      this.show_total = false
    },

    indexOfClosest(nums, target) {
      nums.sort(function(a, b){
        return a.weight - b.weight
      });
      let closest = Number.MAX_SAFE_INTEGER;
      let index = 0;
      nums.forEach((num, i) => {
        let dist = Math.abs(target - num.weight);
        if (dist < closest) {
          index = i;
          closest = dist;
        }
      });
      
      if(nums[index].weight-target<0){
        this.arr_length = nums.length - 1
        index = index+1
        if(index>this.arr_length){
          index = index -1
        }
      }
      return index;
    },
    
    calculate_fee(val){
      if((val > this.item.min_val || this.item.min_val =='false') && (val <= this.item.max_val || this.item.max_val =='false') && val!= ''){
        this.errortext = ''
        this.show_fee = true
        let temp_selling_price = this.selling_price
        this.selling_price = ''
        // this.item.market_fee = (this.item.marketfee_percent/100) * temp_selling_price
        let closing_fee_category = null
        if(temp_selling_price > 0 && temp_selling_price <= 250){
          closing_fee_category = "0-250"
        }
        else if(temp_selling_price > 250 && temp_selling_price <= 500){
          closing_fee_category = "251-500"

        }
        else if(temp_selling_price > 500 && temp_selling_price <= 1000){
          closing_fee_category = "501-1000"
        }
        else if(temp_selling_price > 1000 && temp_selling_price <= 5000){
          closing_fee_category = "1001-5000"
        }
        else{
          closing_fee_category = "5001 and Above"
        }
        let temp = this.closing_fees.find( ({ category }) => category ===  closing_fee_category);
        this.closing_fees.charges = temp.charges
        console.log(this.closing_fees.charges)
        this.selling_price = temp_selling_price
      }
      else if(val == ''){
        this.errortext = "Please enter the selling price first"
      }
      else{
        this.errortext = "Price is too high/low for chosen category. Please choose the correct category for this price."
      }
    },
    with_shipping(){
      this.show_with_shipping = true
    },

    without_shipping(){
      this.show_with_shipping = false
      this.shipping_charges = 0
      this.show_shipping = false
      this.show_total = true
    },

    selected_category(val){
      this.shipping_category = val
    },

    submit(){
      let a = this.indexOfClosest(this.arr, this.weight)
      console.log(this.arr[a])
      this.shipping_charges = this.arr[a].city_price
      if(this.shipping_category == 'city'){
        this.shipping_charges = this.arr[a].city_price
      }

      else if(this.shipping_category == 'state'){
        this.shipping_charges = this.arr[a].state_price
      }

      else if(this.shipping_category == 'rest_of_india'){
        this.shipping_charges = this.arr[a].rest_price
      }

      else if(this.shipping_category == 'metro'){
        this.shipping_charges = this.arr[a].metro_price
      }
      
      else if(this.shipping_category == 'special'){
        this.shipping_charges = this.arr[a].special_price
      }

      this.show_shipping = false
      this.show_total = true
    },

    expand(val){
      if(val == 1){
          this.x = this.options.slice(0,26)
          this.show_product_category = !this.show_product_category 
        
      }
      else if(val == 2){
          this.x = this.options.slice(26,60)
          this.show_product_category = !this.show_product_category
        
      }
      else if(val == 3){
          this.x = this.options.slice(60,63)
          this.show_product_category = !this.show_product_category
      }

      else if(val == 4){
          this.x = this.options.slice(63,76)
          this.show_product_category = !this.show_product_category
      }
      
      else if(val == 5){
          this.x = this.options.slice(76,100)
          this.show_product_category = !this.show_product_category
      }

      else if(val == 6){
          this.x = this.options.slice(100,110)
          this.show_product_category = !this.show_product_category
      }

      else if(val == 7){
          this.x = this.options.slice(110,118)
          this.show_product_category = !this.show_product_category
      }

      else if(val == 8){
          this.x = this.options.slice(118,128)
          this.show_product_category = !this.show_product_category
      }

      else if(val == 9){
          this.x = this.options.slice(128,147)
          this.show_product_category = !this.show_product_category
      }

      else if(val == 10){
          this.x = this.options.slice(147,157)
          this.show_product_category = !this.show_product_category
      }

      else if(val == 11){
          this.x = this.options.slice(157,163)
          this.show_product_category = !this.show_product_category
      }

      else if(val == 12){
          this.x = this.options.slice(163,171)
          this.show_product_category = !this.show_product_category
      }
    },
    
  }
}
</script>

<style>
.outer_container{
  width: 720px;
  background-color: bisque;
  margin:auto;
  margin-bottom: 80px;
  top: 30px;
  position: relative;
  padding: 20px 10px 50px 20px;
  border-radius: 17px;
  box-shadow: 0 4px 8px 0 rgb(0 0 0 / 20%), 20px 19px 20px 0 rgb(0 0 0 / 19%);
}
#autosuggest__input{
  width: 370px;
  border:none;
  border-radius:5px; 
  height: 30px;
}

.first_button{
  padding: 5px 6px;
}

#third_button{
  margin-left: 6px;
}

.inner_container{
  align-items: stretch;
  align-content: flex-start;
  padding:5px;
}

hr.new1 {
  border-top: 2px solid rgb(0, 0, 0);
  margin-top: 0px;
}

.newP{
  font-size: small;
}

.newP2{
  padding-top: 10px;
}

#input_field{
  margin-left: 8px;
}

.shipping{
  font-size: 12px;
  position: absolute;
  left:40px;
  font-weight: 600;
  color: grey;
}

/* .slick-next{

  background-color: aqua;
} */

button.slick-next {
  background-color: red !important;
  margin-right: -100px;
}

button.slick-arrow.slick-prev {
  margin-left: -100px;  
  background-color: red !important;
}

.slick-track{
  width: 3500px;
  height: 200px;
}

.grid-container-shipping{
  display: grid;
  grid-template-columns: auto auto auto auto auto auto;
}

.grid-item-shipping{
  background-color: rgba(255, 255, 255, 0.8);
  border: 1px solid rgba(0, 0, 0, 0.8);
  padding: 05px;
  font-size: 15px;
  text-align: center;
  width:200px;
  margin-left:-169px
}

.grid-container {
  display: grid;
  grid-template-columns: auto auto;
  /* margin-left: 400px; */
}

.grid-item {
  background-color: rgba(255, 255, 255, 0.8);
  border: 1px solid rgba(0, 0, 0, 0.8);
  padding: 05px;
  font-size: 15px;
  text-align: center;
  width:359px;
}

.category{
  display: flex;
  text-align: left;
  margin-left: 400px;
  width: 47.3%;
}

.expand{
  position:absolute; 
  right:410px; 
  padding-top:5px
}

/* .btn-primary{
  width:400px;
  height: 70px;
  margin-bottom:10px;
  margin-right: 20px;
} */

svg:hover {
  cursor: pointer;
  color: red;
}

.modal{
    position: fixed;
    z-index: 1; 
    width: 100%;
    height: 100%; 
    overflow-y:scroll; 
    background-color: rgb(0,0,0); 
    background-color: rgba(0, 0, 0, 0.2);
  }

  .modal-content {
    background-color: #fefefe;
    margin: 10% auto; 
    padding: 20px;
    overflow-y:auto; 
    max-width: 50%; 
  }

.edit{
    position: absolute;
    right:360px;
  }

.edit_shipping{
  position: absolute;
  right:120px
}
</style>
