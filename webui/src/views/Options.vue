<template>
    <v-container class="options">
        <v-row>
            <v-col cols="12" lg="4" class="text-center">
                <ButtonCondition color="success" :condition="isValid && sockedIsConnected" text="Save config" :onclick="save" icon="mdi-content-save" />
            </v-col>
            <v-col cols="12" lg="4" class="text-center">
                <ButtonConfirm color="orange" :condition="sockedIsConnected" text="Wifi Reset" :onclickAgree="wifiReset" disagreeText="Cancel" agreeText="Wifi Reset" title="Wifi Reset!" :cardText="['You are about to delete your WIFI settings, the rest of your settings are not affected!', 'Are you sure you want to continue?']" icon="mdi-wifi-cancel" />
            </v-col>
            <v-col cols="12" lg="4" class="text-center">
                <ButtonConfirm color="red" :condition="sockedIsConnected" text="Factory Reset" :onclickAgree="factoryReset" disagreeText="Cancel" agreeText="Factory Reset" title="Factory Reset!" :cardText="['You are about to delete all your settings, it will also affect the wifi setting!', 'Are you sure you want to continue?']" icon="mdi-harddisk-remove" />
            </v-col>
        </v-row>
        <v-form v-model="isValid">
            <v-row>
                <v-col cols="12" lg="4">
                    <v-card class="pa-1" elevation="4">
                        <v-card-title>
                            <h2>Defaults</h2>
                        </v-card-title>
                        <hr />
                        <br />
                        <v-text-field v-model="config.matrixBrightness" type="number" label="Matrix start brightness" hint="0 to 255" :rules="[rules.required, rules.min0, rules.max255]"></v-text-field>
                        <v-text-field v-model="config.scrollTextDefaultDelay" type="number" label="Scrolltext delay" hint="larger number is slower" suffix="milliseconds" :rules="[rules.required, rules.min0]"></v-text-field>
                        <v-text-field v-model="config.hostname" label="Hostname"></v-text-field>
                        <v-text-field v-model="config.note" label="Note"></v-text-field>
                        <v-switch v-model="config.bootScreenAktiv" label="Bootsceen active" hide-details dense></v-switch>
                    </v-card>
                    <br />
                    <v-card class="pa-2" elevation="4">
                        <v-card-title>
                            <h2>Matrix</h2>
                        </v-card-title>
                        <hr />
                        <br />
                        <v-select :items="matrixTypes" v-model="config.matrixType" label="Matrix type"></v-select>
                        <v-select :items="matrixCorrection" v-model="config.matrixTempCorrection" label="Matrix correction"></v-select>
                    </v-card>
                    <br />
                    <v-card class="pa-2" elevation="4">
                        <v-card-title>
                            <h2>Auto brightness</h2>
                        </v-card-title>
                        <hr />
                        <v-switch v-model="config.matrixBrightnessAutomatic" label="Auto brightness active" dense></v-switch>
                        <v-row>
                            <v-col cols="6" lg="6">
                                <v-text-field v-model="config.mbaDimMin" label="Min bright" hint="0 to 255" type="number" :disabled="!config.matrixBrightnessAutomatic" :rules="config.matrixBrightnessAutomatic ? [rules.required, rules.min0, rules.max255, rules.noDecimals] : []" dense></v-text-field>
                            </v-col>
                            <v-col cols="6" lg="6">
                                <v-text-field v-model="config.mbaDimMax" label="Max bright" hint="0 to 255" type="number" :disabled="!config.matrixBrightnessAutomatic" :rules="config.matrixBrightnessAutomatic ? [rules.required, rules.min0, rules.max255, rules.noDecimals] : []" dense></v-text-field>
                            </v-col>
                        </v-row>
                        <v-row>
                            <v-col cols="6" lg="6">
                                <v-text-field v-model="config.mbaLuxMin" label="From lux" type="number" :disabled="!config.matrixBrightnessAutomatic" :rules="config.matrixBrightnessAutomatic ? [rules.required, rules.min0, rules.noDecimals] : []" dense></v-text-field>
                            </v-col>
                            <v-col cols="6" lg="6">
                                <v-text-field v-model="config.mbaLuxMax" label="To lux" type="number" :disabled="!config.matrixBrightnessAutomatic" :rules="config.matrixBrightnessAutomatic ? [rules.required, rules.min0, rules.noDecimals] : []" dense></v-text-field>
                            </v-col>
                        </v-row>
                    </v-card>
                </v-col>
                <v-col cols="12" lg="4">
                    <v-card class="pa-2" elevation="4">
                        <v-card-title>
                            <h2>Clock</h2>
                        </v-card-title>
                        <hr />
                        <br />
                        <v-text-field v-model="config.ntpServer" label="NTP-Server" hint="domain or ip address" :rules="[rules.required]"></v-text-field>
                        <v-text-field v-model="config.clockTimeZone" type="number" label="UTC offset" hint="your UTC time offset" :rules="[rules.required, rules.minMinus12, rules.max14]"></v-text-field>
                        <ColorPickerTextfield />
                        <v-switch v-model="config.clockDayLightSaving" label="Daylight saving" dense hide-details></v-switch>
                        <v-switch v-model="config.clock24Hours" label="24 Hours" persistent-hint dense hide-details></v-switch>
                        <v-switch v-model="config.clockFatFont" label="Big clock font" persistent-hint dense hide-details></v-switch>
                        <v-switch v-model="config.clockWithSeconds" label="Clock with sek" :disabled="!config.clock24Hours || config.clockFatFont" dense hide-details></v-switch>
                        <v-switch v-model="config.clockSwitchAktiv" label="Switch clock/date active" dense hide-details></v-switch>
                        <v-switch v-model="config.clockBlinkAnimated" label="Flashing time separator" dense hide-details></v-switch>
                        <v-switch v-model="config.clockDayOfWeekFirstMonday" label="Monday as start of the week" dense hide-details></v-switch>
                        <v-switch v-model="config.clockDateDayMonth" label="Date format DD.MM." dense></v-switch>

                        <v-text-field v-model="config.clockSwitchSec" type="number" label="Switch clock/date time" hint="the unit is seconds (s)" suffix="seconds" :disabled="!config.clockSwitchAktiv" :rules="config.clockSwitchAktiv ? [rules.required, rules.min0] : []"></v-text-field>
                        <v-switch v-model="config.clockAutoFallbackActive" label="Clock auto fallback" dense></v-switch>
                        <v-select :items="autoFallbackAnimation" v-model="config.clockAutoFallbackAnimation" label="Fallback Animation" :disabled="!config.clockAutoFallbackActive"></v-select>
                        <v-text-field v-model="config.clockAutoFallbackTime" type="number" label="Fallback time" hint="the unit is seconds (s)" suffix="seconds" :disabled="!config.clockAutoFallbackActive" :rules="config.clockSwitchAktiv ? [rules.required, rules.min0] : []"></v-text-field>
                    </v-card>
                </v-col>
                <v-col cols="12" lg="4">
                    <v-card class="pa-1" elevation="4">
                        <v-card-title>
                            <h2>MQTT</h2>
                        </v-card-title>
                        <hr />
                        <v-switch v-model="config.mqttAktiv" label="MQTT active" dense></v-switch>
                        <v-text-field v-model="config.mqttServer" label="Server" hint="domain or ip address" :disabled="!config.mqttAktiv" :rules="config.mqttAktiv ? [rules.required] : []"></v-text-field>
                        <v-text-field v-model="config.mqttPort" label="Port" type="number" :disabled="!config.mqttAktiv" :rules="config.mqttAktiv ? [rules.required, rules.portRange] : []"></v-text-field>
                        <v-text-field v-model="config.mqttUser" label="User" hint="optional" :disabled="!config.mqttAktiv"></v-text-field>
                        <v-text-field v-model="config.mqttPassword" label="Passsword" hint="optional" :disabled="!config.mqttAktiv"></v-text-field>
                        <v-text-field v-model="config.mqttMasterTopic" label="Master topic" :disabled="!config.mqttAktiv" :rules="config.mqttAktiv ? [rules.required] : []"></v-text-field>
                    </v-card>
                    <br />
                    <v-card class="pa-2" elevation="4">
                        <v-card-title>
                            <h2>Sound</h2>
                        </v-card-title>
                        <hr />
                        <br />
                        <v-text-field v-model="config.initialVolume" type="number" label="Start volume" hint="Between 1 and 30" :rules="[rules.required, rules.volumeRange]"></v-text-field>
                        <v-select :items="pinsESP8266" v-model="config.dfpRXpin" type="number" label="DFPlayer RX pin (ESP8266 only)" :disabled="!config.isESP8266"></v-select>
                        <v-select :items="pinsESP8266" v-model="config.dfpTXpin" type="number" label="DFPlayer TX pin (ESP8266 only)" :disabled="!config.isESP8266"></v-select>
                    </v-card>
                </v-col>
            </v-row>
        </v-form>
    </v-container>
</template>

<script>
import ColorPickerTextfield from "../components/ColorPickerTextfield";
import ButtonCondition from "../components/ButtonCondition";
import ButtonConfirm from "../components/ButtonConfirm";

export default {
    name: "Options",
    data: () => ({
        isValid: true,
    }),
    components: {
        ColorPickerTextfield,
        ButtonCondition,
        ButtonConfirm,
    },
    computed: {
        rules() {
            return this.$store.state.rules;
        },
        config() {
            return this.$store.state.configData;
        },
        sockedIsConnected() {
            return this.$store.state.socket.isConnected;
        },
        matrixTypes() {
            return this.$store.state.matrixTypes;
        },
        matrixCorrection() {
            return this.$store.state.matrixCorrection;
        },
        autoFallbackAnimation() {
            return this.$store.state.autoFallbackAnimation;
        },
        temperatureUnits() {
            return this.$store.state.temperatureUnits;
        },
        ldrDevices() {
            return this.$store.state.ldrDevices;
        },
        pinsESP8266() {
            return this.$store.state.pinsESP8266;
        },
    },
    methods: {
        save() {
            this.$socket.sendObj({ setConfig: this.config });
            setTimeout(() => {
                this.$socket.close();
            }, 3000);
        },
        wifiReset() {
            this.$socket.sendObj({ wifiReset: true });
        },
        factoryReset() {
            this.$socket.sendObj({ factoryReset: true });
        },
        placeHolder() {},
    },
    watch: {
        "$store.state.configData.clock24Hours": function(newVal) {
            if (newVal == false) {
                this.$store.state.configData.clockWithSeconds = newVal;
            }
        },
    },
};
</script>
