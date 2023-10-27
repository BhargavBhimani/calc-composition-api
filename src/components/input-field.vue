<template>
    <div class="flex flex-col w-full justify-around items-center h-[100vh]">
        <div class="flex flex-col justify-center  items-center w-[87vw] rounded-md">
            <input type="text" @input="onKeyboardFunc" v-model="inputString" inputmode="none" @keydown.enter="calculate"
                id="inputField" ref="myInput"
                class="bg-slate-950 w-full h-20 mx-auto outline-none px-5 text-white cursor-pointer text-right text-2xl bottom-0 focus:cursor-text">
            <p class="text-slate-300 text-3xl w-full  flex justify-end">{{ result }}</p>
        </div>
        <div class="grid grid-cols-4 gap-5 ">
            <OperatorButton @click="clearInput">C</OperatorButton>
            <OperatorButton @click="handleFunction('%')" @input="handleFunction">%</OperatorButton>
            <OperatorButton @click="clearLast">
                <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" style="background-color:inherit;"
                    viewBox="0 0 24 24">
                    <path fill="currentColor"
                        d="M19 15.59L17.59 17L14 13.41L10.41 17L9 15.59L12.59 12L9 8.41L10.41 7L14 10.59L17.59 7L19 8.41L15.41 12L19 15.59M22 3a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H7c-.69 0-1.23-.36-1.59-.89L0 12l5.41-8.12C5.77 3.35 6.31 3 7 3h15m0 2H7l-4.72 7L7 19h15V5Z" />
                </svg>
            </OperatorButton>
            <OperatorButton @click="handleFunction('/')" @input="checkInput">/</OperatorButton>
            <TheButton @click="handleFunction(7)">7</TheButton>
            <TheButton @click="handleFunction(8)">8</TheButton>
            <TheButton @click="handleFunction(9)">9</TheButton>
            <OperatorButton @click="handleFunction('*')">X</OperatorButton>
            <TheButton @click="handleFunction(4)">4</TheButton>
            <TheButton @click="handleFunction(5)">5</TheButton>
            <TheButton @click="handleFunction(6)">6</TheButton>
            <OperatorButton @click="handleFunction('-')" class="text-3xl">-</OperatorButton>
            <TheButton @click="handleFunction(1)">1</TheButton>
            <TheButton @click="handleFunction(2)">2</TheButton>
            <TheButton @click="handleFunction(3)">3</TheButton>
            <OperatorButton @click="handleFunction('+')" class="text-3xl">+</OperatorButton>
            <TheButton @click="handleFunction('00')">00</TheButton>
            <TheButton @click="handleFunction(0)">0</TheButton>
            <TheButton @click="handleFunction('.')">.</TheButton>
            <EqlsBtn @click="calculate">=</EqlsBtn>
        </div>
    </div>
</template>

<script setup>
import TheButton from './the-button.vue';
import OperatorButton from './operator-button.vue';
import EqlsBtn from './eqls-button.vue';
import { ref } from 'vue';

const inputString = ref('')
const result = ref('')

const precedence = {
    '+': 2,
    '-': 2,
    '*': 3,
    '/': 3,
    '%': 3,
    '0': 1
}



function handleFunction(val) {
    appendToValue(val);
    checkFirstInput(val);
    checkOperator(val);
    checkInput(val);
    // clearLast(val)
}

function onKeyboardFunc() {
    checkFirstInput();
    checkOperator();
    checkInput();
}

function clearInput() {
    inputString.value = ''
    result.value = ''
}

function appendToValue(val) {
    inputString.value += val
}

function checkOperator() {
    const regex = /[-+*/%]{2,}/g;
    inputString.value = inputString.value.replace(regex, (match) => match[1])
}

function checkInput() {
    inputString.value = inputString.value.trim();
    let pattern = /(\d+(\.\d+)?|\+|\-|\*|\/|\%|\^|\(|\)|\.)|([+\-*/%])\./g;
    let res = inputString.value.match(pattern);
    if (res != null) {
        inputString.value = res.map((match, index) => {
            if (match === '.' && index > 0 && "+-*/%".includes(res[index - 1])) {
                return '0.';
            }
            else if (match === '.' && index < res.length - 1 && /\d/.test(res[index + 1])) {
                return match + '0';
            }
            return match;
        }).join('');
    } else {
        clearInput();
    }
}

function clearLast() {
    const inputField = document.getElementById('inputField');
    let cursorPosition = inputField.selectionStart;
    let val = inputString.value;
    if (cursorPosition > 0) {
        inputString.value = val.slice(0, cursorPosition - 1) + val.slice(cursorPosition, val.length);
        inputField.focus();
        setTimeout(() => {
            inputField.setSelectionRange(cursorPosition - 1, cursorPosition - 1);
        }, 1);
    }
    else {
        inputField.focus();
        setTimeout(() => {
            inputField.setSelectionRange(val.length, val.length);
        }, 1);
    }
}

function checkFirstInput() {
    if (inputString.value.charAt(0) == '.') {
        inputString.value = '0.' + inputString.value.substring(1);
    }
    if (/[0-9\.\-]/.test(inputString.value)) {
        result.value = ''
    } else {
        inputString.value = inputString.value.slice(0, -1);
    }
}

function calculate() {
    try {
        if (inputString.value) {
            const results = evaluateExpression(inputString.value);
            result.value = results;
        } else {
            result.value = 0;
        }
    } catch (error) {
        result.value = "Error";
        console.log(error);
    }
}

function evaluateExpression(expression) {
    let postfixTokens = precedenceLevel(expression);
    let stack = [];

    for (let token of postfixTokens) {
        if (!isNaN(parseFloat(token))) {
            stack.push(parseFloat(token));
        } else {
            let rightOperand = stack.pop();
            let leftOperand = stack.pop();
            if (isNaN(leftOperand)) {
                leftOperand = 0;
            }
            stack.push(evaluateOperation(leftOperand, rightOperand, token));
        }
    }
    return stack.pop();
}

function precedenceLevel(expression) {
    let outputQueue = [];
    let operatorStack = [];

    let tokens = expression.match(/(\d+(\.\d+)?|\+|\-|\*|\/|\(|\)|\[|\]|\%)/g);

    for (let token of tokens) {
        if (!isNaN(parseFloat(token))) {
            outputQueue.push(token);
        } else if (token === '(') {
            operatorStack.push(token);
        } else if (token === ')') {
            while (operatorStack.length > 0 && operatorStack[operatorStack.length - 1] !== '(') {
                outputQueue.push(operatorStack.pop());
            }
            operatorStack.pop();
        } else {
            while (
                operatorStack.length > 0 &&
                precedence[operatorStack[operatorStack.length - 1]] >= precedence[token]
            ) {
                outputQueue.push(operatorStack.pop());
            }
            operatorStack.push(token);
        }
    }

    while (operatorStack.length > 0) {
        outputQueue.push(operatorStack.pop());
    }

    return outputQueue;
}

function evaluateOperation(leftOperand, rightOperand, operator) {
    switch (operator) {
        case '+':
            return leftOperand + rightOperand;
        case '-':
            return leftOperand - rightOperand;
        case '*':
            return leftOperand * rightOperand;
        case '/':
            return leftOperand / rightOperand;
        case '%':
            return leftOperand % rightOperand;
        default:
            throw new Error('Invalid operator: ' + operator);
    }
}


</script>

