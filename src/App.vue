<template>
    <v-app>
        <v-app-bar
                app
                color="primary"
                dark
        >
            Decimal to Binary Converter
        </v-app-bar>

        <v-main>
            <v-main class="grey lighten-3 fill-height">
                <v-container>
                    <v-form
                            style="padding-left: 20%;padding-right: 20%;padding-top: 10%;"
                            ref="form"
                            v-model="valid"
                            lazy-validation
                    >
                        <v-row>
                            <v-slider
                                    v-model="integerBits"
                                    label="Integer bits"
                                    hint="Enter number of bits for integers"
                                    max="64"
                                    min="1"
                                    value="32"
                                    persistent-hint
                                    thumb-label="always"
                                    @input="calculate"
                            ></v-slider>
                            <v-slider
                                    v-model="exponentBits"
                                    label="Exponent bits"
                                    hint="Enter number of exponent bits"
                                    max="64"
                                    min="1"
                                    value="8"
                                    persistent-hint
                                    thumb-label="always"
                                    @input="calculate"
                            ></v-slider>
                            <v-slider
                                    v-model="mantissaBits"
                                    label="Mantissa bits"
                                    hint="Enter number of mantissa bits"
                                    max="64"
                                    min="1"
                                    value="23"
                                    persistent-hint
                                    thumb-label="always"
                                    @input="calculate"
                            ></v-slider>
                        </v-row>
                        <v-row class="pt-12">
                            <v-text-field
                                    v-model="number"
                                    type="number"
                                    label="Enter a number"
                                    placeholder="Eg. 12.45"
                                    outlined
                                    @input="calculate"
                            ></v-text-field>
                        </v-row>
                        <v-row>
                            <v-text-field
                                    v-model="binary"
                                    type="number"
                                    label="Binary number"
                                    id="binary"
                                    placeholder="Binary number will appear here"
                                    outlined
                            ></v-text-field>
                        </v-row>
                        <v-row>
                            <v-text-field
                                    v-model="hex"
                                    type="text"
                                    label="Hex number"
                                    id="hex"
                                    placeholder="Hex number will appear here"
                                    outlined
                            ></v-text-field>
                        </v-row>
                        <v-row class="mt-0">
                            <v-checkbox
                                    v-model="littleEndian"
                                    label="Little endian"
                                    @change="hexChange"
                            ></v-checkbox>
                        </v-row>
                        <v-row>
                            <v-btn
                                    block
                                    color="primary"
                                    x-large
                                    v-clipboard:copy="binary"
                                    v-clipboard:success="onCopy"
                            > {{ copyText }}
                            </v-btn>
                        </v-row>
                    </v-form>
                </v-container>
            </v-main>
        </v-main>
    </v-app>
</template>
<script>
    import Vue from 'vue'
    import VueClipboard from 'vue-clipboard2'

    Vue.use(VueClipboard);
    export default {
        name: 'App',
        created() {
            document.title = 'Decimal to Binary Converter'
        },
        data: () => ({
            integerBits: 32,
            exponentBits: 8,
            mantissaBits: 23,
            copyText: 'Copy to clipboard',
            number: '',
            binary: '',
            binaryArray: [],
            hex: '',
            littleEndian: false
        }),

        methods: {
            calculate() {
                this.copyText = 'Copy to clipboard'
                if (!isNaN(this.number)) {
                    let isFloatingPoint = this.isFloatingPoint(this.number)
                    if (!isFloatingPoint) {
                        this.binaryArray = this.signedToBinary(this.number, this.integerBits)
                    } else {
                        this.binaryArray = this.floatToBinary(this.number)
                    }
                    this.binary = this.binaryArray.join('')
                    this.hex = '0x' + this.binaryToHex(this.binaryArray, this.littleEndian).toUpperCase()
                }
            },
            hexChange() {
                if (!isNaN(this.number)) {
                    this.hex = '0x' + this.binaryToHex(this.binaryArray, this.littleEndian).toUpperCase()
                }
            },
            binaryToHex(binary, isLittleEndian) {
                let hexSize = binary.length / 4;
                let hexValue = 0;
                let hex = []
                for (let i = binary.length - 1, j = 0, k = hexSize - 1; i > -1; i--, j++) {
                    // Divide bits in groups of four and calculate value of them in base 10
                    hexValue += Math.pow(2, j) * binary[i];
                    if (j === 3) {
                        // Find hex char
                        hex[k] = hexValue.toString(16)
                        // Reset indexes
                        j = -1;
                        hexValue = 0;
                        k--;
                    }
                }
                // If need little endian reverse big endian
                if (isLittleEndian) {
                    let tempHex = [];
                    for (let i = 0; i < hexSize; i++) {
                        tempHex[i] = hex[i];
                    }
                    for (let i = 0; i < hexSize; i += 2) {
                        hex[i] = tempHex[hexSize - 2 - i];
                        hex[i + 1] = tempHex[hexSize - 1 - i];
                    }
                }
                return hex.join('')
            },
            signedToBinary(number, bits) {
                let binary = []
                if (number >= 0) {
                    binary = this.unsignedToBinary(number, bits)
                } else {
                    number *= -1
                    binary = this.unsignedToBinary(number, bits)
                    // Flip bits
                    for (let i = 0; i < binary.length; i++) {
                        binary[i] = (binary[i] === 0) ? 1 : 0;
                    }
                    // Add 1
                    for (let i = binary.length - 1; i > 0; i--) {
                        binary[i] = (binary[i] === 0) ? 1 : 0;
                        if (binary[i]) {
                            break;
                        }
                    }
                }
                return binary
            },
            unsignedToBinary(number, bits) {
                let binary = []
                for (let i = bits - 1, j = 0; i >= 0; i--, j++) {
                    let bit = number >> i;
                    binary[j] = bit & 1 ? 1 : 0;
                }
                return binary;
            },
            isFloatingPoint(number) {
                return !(number % 1 === 0);
            },
            getFractionPart(number) {
                if (number < 0) {
                    number *= -1
                }
                return number % 1;
            },
            fractionToBinary(number, numOfBits) {
                let binary = []
                let fractionPart = number
                for (let i = numOfBits - 1, j = 0; i >= 0; i--, j++) {
                    let multiplyFraction = fractionPart * 2.0;
                    if (multiplyFraction >= 1.0) {
                        binary[j] = 1;
                    } else {
                        binary[j] = 0;
                    }
                    fractionPart = this.getFractionPart(multiplyFraction);
                }
                return binary
            },
            findE(integerBinary) {
                return integerBinary.length - 1
            },
            createMantissa(E, fractionBits, integerBinary, fractionBinary, numOfIntBits) {
                let i = 0
                let mantissa = []
                if (E < 0) {
                    for (let j = (E * -1); j < fractionBits; j++, i++) {
                        mantissa[i] = fractionBinary[j];
                    }
                } else {
                    // If E is positive we will start from binary of integer part
                    for (let j = numOfIntBits - E; j < numOfIntBits; j++, i++) {
                        mantissa[i] = integerBinary[j];
                    }
                    for (let j = 0; j < fractionBits; j++, i++) {
                        mantissa[i] = fractionBinary[j];
                    }
                }
                // If empty bits left fill them with zero
                while (i < fractionBits) {
                    mantissa[i] = 0;
                    i++;
                }
                return mantissa
            },
            roundFraction(bitsNeeded, fraction, num, fractionBits) {
                let expanded = 0;
                let i;
                let halfWay = 1;
                if (fraction[num + 1] === 0) {
                    return 0;

                } else {
                    for (i = num + 2; i < bitsNeeded; i++) {
                        if (fraction[i] === 1) {
                            halfWay = 0;
                        }
                    }
                    if (halfWay === 0 && fractionBits >= 0) {
                        for (i = fractionBits; i > 0; i--) {
                            if (fraction[i] === 0) {
                                fraction[i] = 1;
                                return 0;
                            } else {
                                fraction[i] = 0;
                                if (i === 1) {
                                    fraction[i] = 1;
                                    expanded = 1;
                                }
                            }
                        }
                    }
                    if (halfWay === 1 && fractionBits >= 0) {
                        if (fraction[num] === 0)
                            return 0;
                        else {
                            for (i = fractionBits; i > 0; i--) {
                                if (fraction[i] === 0) {
                                    fraction[i] = 1;
                                    return 0;
                                } else {
                                    fraction[i] = 0;
                                    if (i === 1) {
                                        fraction[i] = 1;
                                        expanded = 1;
                                    }
                                }
                            }
                        }
                    }
                }
                return expanded === 1 ? 1 : 0
            },
            calculateBias(numOfExponentBits) {
                return Math.pow(2, numOfExponentBits - 1) - 1;
            },
            floatToBinary(number) {
                let signBit = 0
                let binary = []
                if (number < 0) {
                    signBit = 1
                    number *= -1
                }
                let integerPart = Math.trunc(number)
                // Find how many bits needed for store integer part in base two
                let base2 = Math.log2(number)
                let integerBinaryBits = (integerPart < 1) ? 1 : Math.ceil(base2)
                let fractionPart = this.getFractionPart(number)
                if (fractionPart === 0) {
                    integerBinaryBits++
                }
                // Array for storing bits of integer part
                let integerBinary = this.unsignedToBinary(integerPart, integerBinaryBits)
                let fractionBinary = this.fractionToBinary(fractionPart, this.mantissaBits + 4)
                let E = this.findE(integerBinary)
                let mantissa = this.createMantissa(E, this.mantissaBits, integerBinary, fractionBinary, integerBinaryBits)
                console.log(mantissa)
                this.roundFraction(this.mantissaBits + 2 + integerBinaryBits, mantissa, this.mantissaBits - integerBinaryBits, this.mantissaBits)
                console.log(mantissa)
                // Find how many times point will be floated. Examples:
                // 111.0001 => 1.110001 E=2 | 0.01001 => 1.001 E=-2
                // If E less than zero, we will start mantissa part from fractionBinary[-E]
                // Calculate bias depends on number of exponent bits
                let bias = this.calculateBias(this.exponentBits)
                // Calculate exponent part in base 10. Then we will convert it to binary.
                let e = bias + E;
                if (e === 0) {
                    E = 1 - bias;
                    mantissa = this.createMantissa(E, this.mantissaBits, integerBinary, fractionBinary, integerBinaryBits);
                    this.roundFraction(this.mantissaBits + 2 + integerBinaryBits, mantissa, this.mantissaBits - integerBinaryBits, this.mantissaBits);
                }
                console.log(e)
                console.log(bias)
                console.log(E)
                // Create an array for exponent bits then convert exponent to base 2.
                let exponentBinary = this.unsignedToBinary(e, this.exponentBits);
                console.log(exponentBinary)
                // First bit will be sign bit
                binary[0] = signBit;
                let i = 1;
                // Fill exponent bits
                for (let j = 0; j < this.exponentBits; j++, i++) {
                    binary[i] = exponentBinary[j];
                }
                for (let k = 0; k < this.mantissaBits; k++, i++) {
                    binary[i] = mantissa[k];
                }
                return binary;
            },
            onCopy: function () {
                this.copyText = 'Copied';
            },
        },
    };
</script>
