<template>
  <main>
    <Vodal
      :show="show"
      animation="zoom"
      :closeOnEsc="true"
      :closeButton="true"
      :width="1000"
      :height="500"
      :closeOnClickMask="true"
      v-on:hide="close"
    >
      <v-toolbar>
        <v-icon class="white--text">fa-archive</v-icon>
        <v-toolbar-title class="white--text">
          {{ type == 'view' ? 'Display Drug' : 'Drug Add' }}
        </v-toolbar-title>
      </v-toolbar>
      <v-layout row wrap v-if="type=='add'">
        <v-container fluid class="pb-0">
          <v-tabs v-model="active">
            <v-tabs-bar class="transparent">
              <v-tabs-slider class="active-line"></v-tabs-slider>
              <v-tabs-item v-for="tab in tabs_list" :key="tab" :href="'#' + tab" ripple>
                {{ tab }}
              </v-tabs-item>
            </v-tabs-bar>
            <v-tabs-items>
              <v-tabs-content id="Drug Information" class="my-sub-scroll-enabled">
                <v-form ref="form" fluid class="px-0 pt-3">
                  <v-layout row wrap>
                    <v-flex xs6>
                      <v-card-text class="py-0">
                        <v-select :items="drug_type_cd_list" v-model="detailedInfo['drug_type_cd']" label="*Drug Type:"
                                  @input="changeDrugType"></v-select>
                      </v-card-text>
                    </v-flex>
                    <v-flex xs6 v-if="isAllGroup">
                      <v-card-text class="py-0">
                        <v-select
                          v-bind:items="groupList"
                          v-model="detailedInfo['group_id']"
                          label="Group"
                          item-text="name_txt"
                          item-value="group_id"
                          :rules="[rules.required]"
                          required
                        >
                        </v-select>
                      </v-card-text>
                    </v-flex>
                  </v-layout>
                  <v-layout row wrap>
                    <v-flex xs6>
                      <v-card-text class="py-0">
                        <v-text-field label="NDC" v-model="detailedInfo['ndc']" type="Number"
                                      :disabled="type=='view'" :rules="[rules.required]"
                                      required></v-text-field>
                      </v-card-text>
                    </v-flex>
                    <v-flex xs6>
                      <v-card-text class="py-0">
                        <v-text-field label="Mfg" v-model="detailedInfo['mfg_name_txt']"
                                      :disabled="type=='view'" :rules="[rules.required]"
                                      required></v-text-field>
                      </v-card-text>
                    </v-flex>
                  </v-layout>
                  <v-layout row wrap>
                    <v-flex xs6>
                      <v-card-text class="py-0">
                        <v-text-field label="Drug Name" v-model="detailedInfo['product_name_txt']"
                                      :disabled="type=='view'" :rules="[rules.required]"
                                      required></v-text-field>
                      </v-card-text>
                    </v-flex>
                    <v-flex xs6>
                      <v-card-text class="py-0">
                        <v-text-field label="Additional Info" v-model="detailedInfo['additional_desc_txt']"
                                      :disabled="type=='view'"></v-text-field>
                      </v-card-text>
                    </v-flex>
                    <v-flex xs6>
                      <v-card-text class="py-0">
                        <v-select
                          v-bind:items="productTypeList"
                          v-model="detailedInfo['product_type_cd']"
                          label="Product Type"
                          :rules="[rules.required]"
                          required
                          bottom>
                        </v-select>
                      </v-card-text>
                    </v-flex>
                    <v-flex xs6>
                      <v-card-text class="py-0">
                        <v-select
                          v-bind:items="$store.state.drugSetupData && $store.state.drugSetupData.roa_list || []"
                          v-model="detailedInfo['roa_cd']"
                          label="ROA"
                          :rules="[rules.required]"
                          required
                          item-text="Description"
                          item-value="Code"
                        >
                          <template slot="selection" slot-scope="data">
                            {{ data.item.Code }} - {{ data.item.Description }}
                          </template>
                          <template slot="item" slot-scope="data">
                            {{ data.item.Code }} - {{ data.item.Description }}
                          </template>
                        </v-select>
                      </v-card-text>
                    </v-flex>
                  </v-layout>
                  <v-layout row wrap>
                    <v-flex xs6>
                      <v-card-text class="py-0">
                        <v-select
                          v-bind:items="dosageList"
                          v-model="detailedInfo['form_cd']"
                          label="Dosage"
                          :rules="[rules.required]"
                          required
                        >
                        </v-select>
                      </v-card-text>
                    </v-flex>
                    <v-flex xs6>
                      <v-card-text class="py-0">
                        <v-select
                          v-bind:items="deaClassList"
                          v-model="detailedInfo['dea_class_cd']"
                          label="DEA Class"
                          :rules="[rules.required]"
                          required
                        >
                        </v-select>
                      </v-card-text>
                    </v-flex>
                    <v-flex xs6>
                      <v-card-text class="py-0">
                        <v-text-field label="Strength:" v-model="detailedInfo['strength_txt']"
                                      :disabled="type=='view'"></v-text-field>
                      </v-card-text>
                    </v-flex>
                  </v-layout>
                  <v-layout row wrap>
                    <v-flex xs6>
                      <v-card-text>
                        <p>Source</p>
                        <v-divider></v-divider>
                        <v-layout row wrap>
                          <v-flex xs12>
                            <v-radio-group v-model="detailedInfo['single_src_ind']" row>
                              <v-radio label="Single Source" value="Y" :disabled="type=='view'"></v-radio>
                              <v-radio label="Multi-Source" value="N" :disabled="type=='view'"></v-radio>
                            </v-radio-group>
                          </v-flex>
                        </v-layout>
                      </v-card-text>
                    </v-flex>

                    <v-flex xs6>
                      <v-card-text>
                        <p>Form</p>
                        <v-divider></v-divider>
                        <v-layout row wrap>
                          <v-flex xs12>
                            <v-radio-group v-model="detailedInfo['form_type_cd']" row>
                              <v-radio label="Solid" value="S" :disabled="type=='view'"></v-radio>
                              <v-radio label="Liquid" value="L" :disabled="type=='view'"></v-radio>
                            </v-radio-group>
                          </v-flex>
                        </v-layout>
                      </v-card-text>
                    </v-flex>

                  </v-layout>
                  <v-layout row wrap>
                    <v-flex xs6>
                      <v-card-text>
                        <p>Type</p>
                        <v-divider></v-divider>
                        <v-layout row wrap>
                          <v-flex xs12>
                            <v-radio-group v-model="detailedInfo['maint_drug_ind']" row>
                              <v-radio label="Maintenance Med" value="Y" :disabled="type=='view'"></v-radio>
                              <v-radio label="Acute Med" value="N" :disabled="type=='view'"></v-radio>
                            </v-radio-group>
                          </v-flex>
                        </v-layout>
                      </v-card-text>
                    </v-flex>

                    <v-flex xs6>
                      <v-card-text>
                        <p>Unit of Measure</p>
                        <v-divider></v-divider>
                        <v-layout row wrap>
                          <v-flex xs12>
                            <v-radio-group v-model="detailedInfo['std_uom_cd']" row>
                              <v-radio label="EA" value="EA" :disabled="type=='view'"></v-radio>
                              <v-radio label="ML" value="ML" :disabled="type=='view'"></v-radio>
                              <v-radio label="GM" value="GM" :disabled="type=='view'"></v-radio>
                            </v-radio-group>
                          </v-flex>
                        </v-layout>
                      </v-card-text>
                    </v-flex>
                  </v-layout>

                  <v-layout row>
                    <v-card-text>
                      <p>Unit price / Price Effective Date</p>
                      <v-divider></v-divider>
                      <v-layout row wrap>
                        <v-flex xs3>
                          <v-card-text>
                            <p>AWP</p>
                            <v-text-field label="Unit Price" v-model="detailedInfo['awp_curr_unit_price_amt']"
                                          :disabled="type=='view'" type="number"></v-text-field>
                            <date-picker
                              v-model="detailedInfo['awp_curr_price_effect_dt']"
                              label="Price Effective Date"
                            ></date-picker>
                          </v-card-text>
                        </v-flex>

                        <v-flex xs3>
                          <v-card-text>
                            <p>Direct</p>
                            <v-text-field label="Unit Price" v-model="detailedInfo['dp_curr_unit_price_amt']"
                                          :disabled="type=='view'" type="number"></v-text-field>
                            <date-picker
                              v-model="detailedInfo['dp_curr_price_effect_dt']"
                              label="Price Effective Date"
                            ></date-picker>
                          </v-card-text>
                        </v-flex>

                        <v-flex xs3>
                          <v-card-text>
                            <p>HFCA</p>
                            <v-text-field label="Unit Price" v-model="detailedInfo['mac_curr_unit_price_amt']"
                                          :disabled="type=='view'" type="number"></v-text-field>
                            <date-picker
                              v-model="detailedInfo['mac_curr_price_effect_dt']"
                              label="Price Effective Date"
                            ></date-picker>
                          </v-card-text>
                        </v-flex>

                        <v-flex xs3>
                          <v-card-text>
                            <p>WAC</p>
                            <v-text-field label="Unit Price" v-model="detailedInfo['wac_curr_unit_price_amt']"
                                          :disabled="type=='view'" type="number"></v-text-field>
                            <date-picker
                              v-model="detailedInfo['wac_curr_price_effect_dt']"
                              label="Price Effective Date"
                            ></date-picker>
                          </v-card-text>
                        </v-flex>

                      </v-layout>
                    </v-card-text>
                  </v-layout>
                </v-form>
              </v-tabs-content>

              <v-tabs-content id="Compound Information" class="my-sub-scroll-enabled">
                <v-layout row wrap>
                  <v-flex xs6>
                    <v-card-text>
                      <p>HCPCS</p>
                      <v-divider></v-divider>
                      <v-layout row wrap>
                        <v-card-text class="py-0">
                          <v-text-field label="HCPCS" v-model="detailedInfo['hcpcs']"></v-text-field>
                        </v-card-text>
                        <v-card-text class="py-0">
                          <v-text-field label="Mod 1" v-model="detailedInfo['hcpcsMod1']"></v-text-field>
                        </v-card-text>
                        <v-card-text class="py-0">
                          <v-text-field label="Mod 2" v-model="detailedInfo['hcpcsMod2']"></v-text-field>
                        </v-card-text>
                        <v-card-text class="py-0">
                          <v-text-field label="Mod 3" v-model="detailedInfo['hcpcsMod3']"></v-text-field>
                        </v-card-text>
                      </v-layout>

                      <v-layout row>
                        <v-flex xs12>
                          <v-card-text class="py-0">
                            <date-picker
                              label="Start Date"
                              v-model="detailedInfo['hcpcsStartDt']"
                            >
                            </date-picker>
                          </v-card-text>
                        </v-flex>
                      </v-layout>
                      <v-layout row>
                        <v-flex xs12>
                          <v-card-text class="hasEndDate-container">
                            <v-checkbox label="Has an end date?" v-model="hasEndDate"></v-checkbox>
                          </v-card-text>
                        </v-flex>
                      </v-layout>

                      <v-layout row v-if="hasEndDate">
                        <v-flex xs12>
                          <v-card-text class="py-0">
                            <date-picker
                              label="End Date"
                              v-model="detailedInfo['hcpcsEndDt']"
                            >
                            </date-picker>
                          </v-card-text>
                        </v-flex>
                      </v-layout>
                    </v-card-text>
                  </v-flex>
                  <v-flex xs6>
                    <v-card-text>
                      <p>Age Limits</p>
                      <v-divider></v-divider>
                      <v-layout row wrap>
                        <v-flex xs6>
                          <v-card-text class="py-0">
                            <v-text-field label="Minimum Age" v-model="detailedInfo['minAge']"
                                          type='number'></v-text-field>
                          </v-card-text>
                        </v-flex>
                        <v-flex xs6>
                          <v-card-text class="py-0">
                            <v-text-field label="Maximum Age" v-model="detailedInfo['maxAge']"
                                          type='number'></v-text-field>
                          </v-card-text>
                        </v-flex>
                      </v-layout>
                    </v-card-text>

                    <v-card-text>
                      <p>Quantity Limits</p>
                      <v-divider></v-divider>
                      <v-layout row wrap>
                        <v-flex xs6>
                          <v-card-text class="py-0">
                            <v-text-field label="Minimum Quantity" v-model="detailedInfo['minQty']"
                                          type='number'></v-text-field>
                          </v-card-text>
                        </v-flex>
                        <v-flex xs6>
                          <v-card-text class="py-0">
                            <v-text-field label="Maximum Quantity" v-model="detailedInfo['maxQty']"
                                          type='number'></v-text-field>
                          </v-card-text>
                        </v-flex>
                      </v-layout>
                    </v-card-text>

                    <v-card-text>
                      <p>External Requirements</p>
                      <v-divider></v-divider>
                      <v-layout row wrap>
                        <v-flex xs12>
                          <v-card-text class="pb-0">
                            <v-checkbox label="Documentation Required"
                                        v-model="detailedInfo['invoiceInd']"></v-checkbox>
                          </v-card-text>
                        </v-flex>
                        <v-flex xs12>
                          <v-card-text class="py-0">
                            <v-checkbox label="Requires a Prior Authorization"
                                        v-model="detailedInfo['priorAuthInd']"></v-checkbox>
                          </v-card-text>
                        </v-flex>
                      </v-layout>
                    </v-card-text>
                  </v-flex>
                  <v-flex xs12>
                    <v-layout>
                      <v-flex xs6>
                        <v-card-text>
                          <p>Quantity Pricing</p>
                          <v-divider></v-divider>

                          <v-card-text class="pb-0">
                            <v-checkbox label="Use Quantity Pricing"
                                        v-model="useQuantityPricing"></v-checkbox>
                          </v-card-text>

                          <v-card-text class="py-0" v-if="useQuantityPricing">
                            <v-text-field label="Quantity" v-model="detailedInfo['quantity']"
                                          type="number"></v-text-field>
                          </v-card-text>
                          <v-card-text class="py-0" v-else>
                            <v-text-field label="Quantity" v-model="detailedInfo['quantity']" type="number"
                                          disabled></v-text-field>
                          </v-card-text>
                        </v-card-text>
                      </v-flex>
                      <v-flex xs6>
                        <v-card-text>
                          <p>Quantity Conversion</p>
                          <v-divider></v-divider>
                          <v-card-text class="pb-0">
                            <v-checkbox label="Requires a Quantity Conversation"
                                        v-model="requireQuantityConversation"></v-checkbox>
                          </v-card-text>

                          <v-card-text class="py-0" v-if="requireQuantityConversation">
                            <v-text-field label="Qty Conversion Factor" v-model="detailedInfo['conversionFactor']"
                                          type='number'></v-text-field>
                          </v-card-text>
                          <v-card-text class="py-0" v-else>
                            <v-text-field label="Qty Conversion Factor" v-model="detailedInfo['conversionFactor']"
                                          type='number' disabled></v-text-field>
                          </v-card-text>
                        </v-card-text>
                      </v-flex>
                    </v-layout>
                  </v-flex>
                </v-layout>
                <v-layout>
                  <v-card-text class="py-0">
                    <v-text-field label="Local use" v-model="detailedInfo['localUse']"></v-text-field>
                  </v-card-text>
                </v-layout>
              </v-tabs-content>
            </v-tabs-items>
          </v-tabs>
        </v-container>
      </v-layout>
      <v-container fluid class="my-sub-scroll-enabled px-0" v-else>
        <v-layout row wrap>
          <v-flex xs6>
            <v-card-text class="py-0">
              <v-text-field label="NDC:" v-model="info['NDC']" type="Number" :disabled="type=='view'"></v-text-field>
            </v-card-text>
          </v-flex>
          <v-flex xs6>
            <v-card-text class="py-0">
              <v-text-field label="Mfg:" v-model="info['Mfg']" :disabled="type=='view'"></v-text-field>
            </v-card-text>
          </v-flex>
        </v-layout>
        <v-layout row wrap>
          <v-flex xs6>
            <v-card-text class="py-0">
              <v-text-field label="Drug Name:" v-model="info['Product Name']" :disabled="type=='view'"></v-text-field>
            </v-card-text>
          </v-flex>
          <v-flex xs6>
            <v-card-text class="py-0">
              <v-text-field label="Additional Info:" v-model="detailedInfo['additional_desc_txt']"
                            :disabled="type=='view'"></v-text-field>
            </v-card-text>
          </v-flex>
          <v-flex xs12>
            <v-card-text class="py-0">
              <v-text-field label="Product Type:" v-model="info['Product Type Desc']"
                            :disabled="type=='view'"></v-text-field>
            </v-card-text>
          </v-flex>
        </v-layout>
        <v-layout row wrap>
          <v-flex xs6>
            <v-card-text class="py-0">
              <v-text-field label="ROA:" v-model="info['ROA']" :disabled="type=='view'"></v-text-field>
            </v-card-text>
          </v-flex>
          <v-flex xs6>
            <v-card-text class="py-0">
              <v-text-field label="DEA Class:" v-model="detailedInfo['dea_class_cd']"
                            :disabled="type=='view'"></v-text-field>
            </v-card-text>
          </v-flex>
        </v-layout>
        <v-layout row wrap>
          <v-flex xs6>
            <v-card-text class="py-0">
              <v-text-field label="Dosage:" v-model="detailedInfo['form_cd']" :disabled="type=='view'"></v-text-field>
            </v-card-text>
          </v-flex>
          <v-flex xs6>
            <v-card-text class="py-0">
              <v-text-field label="Package Size:" v-model="detailedInfo['std_pkg_size']"
                            :disabled="type=='view'"></v-text-field>
            </v-card-text>
          </v-flex>
        </v-layout>
        <v-layout row wrap>
          <v-flex xs6>
            <v-card-text class="py-0">
              <v-text-field label="Strength:" v-model="detailedInfo['strength_txt']"
                            :disabled="type=='view'"></v-text-field>
            </v-card-text>
          </v-flex>
        </v-layout>
        <v-layout row wrap>
          <v-flex xs6>
            <v-card-text>
              <p>Source</p>
              <v-divider></v-divider>
              <v-layout row wrap>
                <v-flex xs12>
                  <v-radio-group v-model="detailedInfo['single_src_ind']" row>
                    <v-radio label="Single Source" value="Y" :disabled="type=='view'"></v-radio>
                    <v-radio label="Multi-Source" value="N" :disabled="type=='view'"></v-radio>
                  </v-radio-group>
                </v-flex>
              </v-layout>
            </v-card-text>
          </v-flex>

          <v-flex xs6>
            <v-card-text>
              <p>Form</p>
              <v-divider></v-divider>
              <v-layout row wrap>
                <v-flex xs12>
                  <v-radio-group v-model="detailedInfo['form_type_cd']" row>
                    <v-radio label="Solid" value="S" :disabled="type=='view'"></v-radio>
                    <v-radio label="Liquid" value="L" :disabled="type=='view'"></v-radio>
                  </v-radio-group>
                </v-flex>
              </v-layout>
            </v-card-text>
          </v-flex>

        </v-layout>
        <v-layout row wrap>
          <v-flex xs6>
            <v-card-text>
              <p>Type</p>
              <v-divider></v-divider>
              <v-layout row wrap>
                <v-flex xs12>
                  <v-radio-group v-model="detailedInfo['maint_drug_ind']" row>
                    <v-radio label="Maintenance Med" value="Y" :disabled="type=='view'"></v-radio>
                    <v-radio label="Acute Med" value="N" :disabled="type=='view'"></v-radio>
                  </v-radio-group>
                </v-flex>
              </v-layout>
            </v-card-text>
          </v-flex>

          <v-flex xs6>
            <v-card-text>
              <p>Unit of Measure</p>
              <v-divider></v-divider>
              <v-layout row wrap>
                <v-flex xs12>
                  <v-radio-group v-model="detailedInfo['std_uom_cd']" row>
                    <v-radio label="EA" value="EA" :disabled="type=='view'"></v-radio>
                    <v-radio label="ML" value="ML" :disabled="type=='view'"></v-radio>
                    <v-radio label="GM" value="GM" :disabled="type=='view'"></v-radio>
                  </v-radio-group>
                </v-flex>
              </v-layout>
            </v-card-text>
          </v-flex>
        </v-layout>
        <v-layout row>
          <v-card-text>
            <p>Unit price / Price Effective Date</p>
            <v-divider></v-divider>
            <v-layout row wrap>
              <v-flex xs3>
                <v-card-text class="pb-0">
                  <p>AWP</p>
                  <v-text-field label="Unit Price" v-model="detailedInfo['awp_curr_unit_price_amt']"
                                :disabled="type=='view'"></v-text-field>
                  <v-text-field label="Price Effective Date" v-model="detailedInfo['awp_curr_price_effect_dt']"
                                :disabled="type=='view'" hide-details></v-text-field>
                </v-card-text>
              </v-flex>

              <v-flex xs3>
                <v-card-text class="pb-0">
                  <p>Direct</p>
                  <v-text-field label="Unit Price" v-model="detailedInfo['dp_curr_unit_price_amt']"
                                :disabled="type=='view'"></v-text-field>
                  <v-text-field label="Price Effective Date" v-model="detailedInfo['dp_curr_price_effect_dt']"
                                :disabled="type=='view'" hide-details></v-text-field>
                </v-card-text>
              </v-flex>

              <v-flex xs3>
                <v-card-text class="pb-0">
                  <p>HFCA</p>
                  <v-text-field label="Unit Price" v-model="detailedInfo['mac_curr_unit_price_amt']"
                                :disabled="type=='view'"></v-text-field>
                  <v-text-field label="Price Effective Date" v-model="detailedInfo['mac_curr_price_effect_dt']"
                                :disabled="type=='view'" hide-details></v-text-field>
                </v-card-text>
              </v-flex>

              <v-flex xs3>
                <v-card-text class="pb-0">
                  <p>WAC</p>
                  <v-text-field label="Unit Price" v-model="detailedInfo['wac_curr_unit_price_amt']"
                                :disabled="type=='view'"></v-text-field>
                  <v-text-field label="Price Effective Date" v-model="detailedInfo['wac_curr_price_effect_dt']"
                                :disabled="type=='view'" hide-details></v-text-field>
                </v-card-text>
              </v-flex>
            </v-layout>
          </v-card-text>
        </v-layout>
      </v-container>
      <v-divider></v-divider>
      <v-card-actions class="d-block text-xs-center">
        <v-btn v-if="type=='add'" color="primary-color" @click.native="save()">Save</v-btn>
        <v-btn @click.native="close">Close</v-btn>
      </v-card-actions>
    </Vodal>
  </main>
</template>

<script>
  import Vodal from '../../../components/common/Vodal.vue'
  import drugServices from '../../../services/drug/drugServices'
  import claimServices from '../../../services/claim/claimServices'
  import {
    convertStringToPickerFormat,
    stringToDate
  } from '../../../components/utils/DateMethods'
  import store from '../../../store'
  import DatePicker from "../../../components/common/DatePicker.vue"

  const dosageList = [
    "ACC - Accessory",
    "AER - Aerosol Liquid",
    "ARO - Aerosol Powder",
    "BAN - Bandage",
    "BAR - Bar",
    "BEA - Beads",
    "C12 - Capsule, Extended Release, 12 HR",
    "C24 - Capsule, Extended Release, 24 HR",
    "CAK - Cake",
    "CAP - Capsule",
    "CER - Capsule, Extended Release",
    "CHI - Chip",
    "CRE - Cream",
    "CRY - Crystal",
    "CTB - Tablet, Chewable",
    "DEV - Device",
    "DRE - Dressing",
    "DSK - Disk",
    "ECC - Capsule, Delayed Release",
    "ECT - Tablet, Enteric Coated",
    "ELI - Elixir",
    "EMO - Emollient Cream",
    "EMU - Emulsion",
    "FDS - Food, Solid",
    "FIL - Film",
    "FLA - Flake",
    "FOA - Foam",
    "GAS - Gas",
    "GEF - Powder, Effervescent",
    "GEL - Gel/Jelly",
    "GER - Powder for Suspension, Extended Release",
    "GFS - Gel Forming Solution",
    "GRA - Granule",
    "GUM - Gum",
    "ICR - Insert, Extended Release",
    "IMP - Implant",
    "INJ - Injectable",
    "KIT - Kit",
    "LEA - Leaf",
    "LIQ - Liquid",
    "LOT - Lotion",
    "LOZ - Lozenge/Troche",
    "LUM - Lump",
    "NMA - Enema",
    "ODT - Tablet, Disintegrating",
    "OIL - Oil",
    "OIN - Ointment",
    "PAD - Pad",
    "PAS - Paste",
    "PDR - Powder for Suspension",
    "PDS - Powder for Solution",
    "PEL - Pellet",
    "PI1 - Powder for Suspension, 1 Month",
    "PI3 - Powder for Suspension, 3 Month",
    "PI4 - Powder for Suspension, 4 Month",
    "PKT - Packet",
    "POD - Pod",
    "POW - Powder",
    "PRO - Prophylactic",
    "PUD - Pudding",
    "SER - Suspension, Extended Release",
    "SGL - Capsule, Liquid Filled",
    "SHA - Shampoo",
    "SHE - Sheet",
    "SOA - Soap",
    "SOL - Solution",
    "SPE - Suppository, Extended Release",
    "SPG - Sponge",
    "SPR - Spray",
    "STI - Stick",
    "SUP - Suppository",
    "SUS - Suspension",
    "SWA - Swab",
    "SYR - Syrup",
    "T12 - Tablet, Extended Release, 12 HR",
    "T24 - Tablet, Extended Release, 24 HR",
    "TAB - Tablet",
    "TAM - Tampon",
    "TAP - Tape",
    "TCP - Tablet, Delayed Release",
    "TDM - Patch, Extended Release",
    "TEF - Tablet, Effervescent",
    "TER - Tablet, Extended Release",
    "TES - Test",
    "TIN - Tincture",
    "WAF - Wafer",
    "WAX - Wax"
  ]
  const productTypeList = [
    "01 - Rx/Brand Name",
    "02 - Rx/Generic",
    "03 - Surgical/Device",
    "04 - Rx/Repackaged Brand Name Product",
    "05 - Rx/Branded Generic",
    "07 - OTC/Brand Name",
    "08 - OTC/Generic",
    "09 - OTC/Repackaged",
    "10 - Rx/Repackaged Branded Generic Product",
    "11 - Chemical for Compounding",
    "FE - Fee",
    "IC - Ingredient Cost From Pharmacy",
    "KI - Kit",
    "CF - Compound Drug (Formulary)",
    "CN - Compound Drug (Non-Formulary)",
    "MC - Drug that uses MAC table for Pricing",
    "SB - Use Submitted Price From Pharmacy"
  ]
  const deaClassList = [
    "RX",
    "OTC",
    "CI",
    "CII",
    "CIII",
    "CIV",
    "CV"
  ]

  const defaultDetailedInfo = {
    drug_type_cd: 'C',
    group_id: 0,
    ndc: null,
    mfg_name_txt: null,
    single_src_ind: 'Y',
    form_type_cd: 'S',
    maint_drug_ind: 'Y',
    std_uom_cd: 'EA',
    wac_curr_unit_price_amt: '0',
    awp_curr_unit_price_amt: '0',
    dp_curr_unit_price_amt: '0',
    mac_curr_unit_price_amt: '0',
    wac_curr_price_effect_dt: new Date().toLocaleDateString(),
    awp_curr_price_effect_dt: new Date().toLocaleDateString(),
    dp_curr_price_effect_dt: new Date().toLocaleDateString(),
    mac_curr_price_effect_dt: new Date().toLocaleDateString(),
    hcpcsStartDt: new Date().toLocaleDateString(),
    hcpcsEndDt: '9998-12-31'
  }

  export default {
    name: 'drug-detail-form',
    props: ['data', 'type', 'show', 'closeForm', 'saveDrug'],
    components: {
      DatePicker,
      Vodal
    },
    data() {
      return {
        tabs_list: ['Drug Information'],
        drug_type_cd_list: [{
          value: 'S',
          text: 'Special'
        }, {
          text: 'Compound',
          value: 'C'
        }],
        info: {},
        productTypeList: productTypeList,
        dosageList: dosageList,
        deaClassList: deaClassList,
        groupList: [],
        active: '',
        drugFileType: 0,
        detailedInfo: {...defaultDetailedInfo},
        hasEndDate: false,
        useQuantityPricing: false,
        requireQuantityConversation: false,
        isAllGroup: false,
        rules: {
          required: (value) => !!value || value == 0 || 'Required.'
        }
      }
    },
    watch: {
      data: {
        handler: function () {
          if (this.data) {
            this.info = this.data
            this.detailedInfo = Object.assign({}, {...defaultDetailedInfo}, this.data)
            this.setValue()
          }
        },
        deep: true
      },
      show: function (val) {
        if (val && this.type === 'add') {
          this.detailedInfo = {...defaultDetailedInfo}
          this.setValue()
          this.tabs_list = ['Drug Information', 'Compound Information']
          this.isAllGroup = true
        }
      },
    },
    mounted() {
      const data = {
        search: {
          Pcc: store.state.pcn.pcc
        },
        orderBy: 'Name',
        includeProperties: ''
      }

      const that = this
      claimServices.getGroupsList(data).then(function (groups) {
        that.groupList = []
        if (groups.length > 0) {
          groups.forEach(function (d) {
            that.groupList.push({name_txt: d.Code + '-' + d.Name,  group_id: d.Id})
          })
        }
        that.groupList.unshift({group_id: 0, name_txt: "All Groups"})
      })
    },
    methods: {
      close: function () {
        this.closeForm()
      },
      setValue: function () {
        const that = this
        if (this.info && this.info.hasOwnProperty('NDC') && this.info['NDC']) {
          const data = {
            "ndc": this.info['NDC']
          }
          drugServices.getDrugDataWthDnc(data).then(function (v) {
            if (v.drug_data.length > 0) {
              const temp = v.drug_data[0]
              that.detailedInfo = Object.assign({},
                {...that.detailedInfo},
                {...temp},
                {
                  awp_curr_price_effect_dt: convertStringToPickerFormat(temp['awp_curr_price_effect_dt']),
                  dp_curr_price_effect_dt: convertStringToPickerFormat(temp['dp_curr_price_effect_dt']),
                  mac_curr_price_effect_dt: convertStringToPickerFormat(temp['mac_curr_price_effect_dt']),
                  wac_curr_price_effect_dt: convertStringToPickerFormat(temp['wac_curr_price_effect_dt'])
                })
            }
          })
        }
      },
      save: function () {
        const newInfo = Object.assign({}, {...this.detailedInfo}, {
          "form_cd": this.detailedInfo['form_cd'] && this.detailedInfo['form_cd'].split('-')[0].trim(),
          "product_type_cd": this.detailedInfo['product_type_cd'] && this.detailedInfo['product_type_cd'].split('-')[0].trim(),
          hcpcsStartDt: stringToDate(this.detailedInfo['awp_curr_price_effect_dt'], 'yyyy-mm-dd', '-', 'date').toLocaleDateString(),
          hcpcsEndDt: stringToDate(this.detailedInfo['awp_curr_price_effect_dt'], 'yyyy-mm-dd', '-', 'date').toLocaleDateString()
        })
        const that = this
        if (this.$refs.form.validate()) {
          drugServices.addDrug(newInfo, this.$store.state.user, this.$store.state.pcn.pcc).then(function (v) {
            if (v && (v[0] === '0' || v[0] === null)) {
              that.selectedDrug = {}
              that.close()
              window.Vue.$emit('snackbar', 'success', 'Added Drug successfully!');
            }
          })
        } else {
          this.active = 'Drug Information'
        }
      },
      changeDrugType: function (value) {
        if (value === 'C') {
          this.tabs_list = ['Drug Information', 'Compound Information']
          this.isAllGroup = true
        } else {
          this.tabs_list = ['Drug Information']
          this.isAllGroup = false
        }
      }
    }
  }
</script>

<style lang="stylus">
</style>
