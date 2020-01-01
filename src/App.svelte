<script>
  import Button from "./Button.svelte";

  const operatorMap = ["+", "-", "*", "/", "="];
  const numericalMap = ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "."];

  let equalsPressed = false;
  let expression = "";
  let expressionEquals = "";
  let value = "0";

  let screenContainerWidth;
  let equalsWidth;
  let expWidth;
  let margin = 4;
  let maxCharWidth = 0;

  function isValidExpression(exp) {
    return /^(?:\d+[+\-*/]|\d+\.\d+[+\-*/]|\-\d+[+\-*/]|\.\d+[+\-*/])+(?:\.\d+|\d+|\d+\.\d+)$/.test(
      exp
    );
  }

  function onNumericalKeyPress(v) {
    value === "0" ? (value = v) : (value += v);
    expression += v;
  }

  function clearEqualsPressed() {
    if (equalsPressed) {
      equalsPressed = false;
    }
  }

  function onBackKeyPress() {
    clearEqualsPressed();
    if (expression.length >= 1) {
      expression = expression.substring(0, expression.length - 1);
    }

    if (value.length > 1) {
      value = value.substring(0, value.length - 1);
    } else if (value.length === 1 && value !== "0") {
      value = "0";
    }
  }

  function evaluateExpression() {
    //avoiding eval by using function constructor
    let ev = new Function(`return ${expression}`);
    expressionEquals = ev();
  }

  function onEqualsPress() {
    if (isValidExpression(expression)) {
      equalsPressed = true;
      evaluateExpression();
    }
  }

  function onClearKeyPress() {
    value = "0";
    expression = "";
    expressionEquals = "";
    clearEqualsPressed();
  }

  function handleKeyPresses(event) {
    if (operatorMap.includes(event.key)) {
      onOperatorKeyPress(event.key);
    } else if (numericalMap.includes(event.key)) {
      onNumericalKeyPress(event.key);
    } else if (event.key === "Backspace") {
      onBackKeyPress();
    } else if (event.key === "Escape") {
      onClearKeyPress();
    } else if (event.key === "Enter") {
      onEqualsPress();
    }
  }

  function onOperatorKeyPress(operator) {
    clearEqualsPressed();

    if (value === "0" && operator === "-") {
      value = operator;
      expression += operator;
    } else if (value !== "0" && !lastCharIsOperator) {
      expression += operator;
      value = "0";
    }
  }

  $: expressionWidth = !!equalsWidth
    ? expWidth - margin + (equalsWidth - margin)
    : expWidth - margin;

  $: {
    if (expressionWidth >= screenContainerWidth && maxCharWidth === 0) {
      maxCharWidth = expression.length;
      console.log("MAX CHARS", maxCharWidth);
    } else {
      maxCharWidth = 0;
    }
  }

  $: lastCharIsOperator = operatorMap.includes(
    expression[expression.length - 1]
  );

  $: roundedEquals = /\./.test(expressionEquals)
    ? Number(expressionEquals).toFixed(2)
    : expressionEquals;

  $: isValidExpression(expression)
    ? evaluateExpression()
    : (expressionEquals = "");

  $: expressionWithEntities = expression.replace(
    /(?:\*)|(?:\/)|(?:\+)|(?:\-)/g,
    match => {
      if (match === "*") {
        return " &times; ";
      } else if (match === "/") {
        return " &divide; ";
      } else if (match === "+") {
        return " &plus; ";
      } else if (match === "-") {
        return " &minus; ";
      } else {
        return match;
      }
    }
  );
</script>

<style>
  h4 {
    color: #808090;
  }

  .container {
    display: flex;
    flex-flow: row nowrap;
    width: 100%;
    height: 100%;
    background: #c1deba;
    justify-content: center;
    align-items: center;
  }

  main {
    display: grid;
    grid-template-rows: 85px 1fr;
    grid-gap: 10px;
    width: 550px;
    height: 700px;
    background: #cdd1d4;
    border: 2px solid #808090;
    margin-top: -40px;
    padding: 10px;
    /* box-shadow: 0px 5px 1px 2px #808090; */
    box-shadow: 1px 2px 9px 2px #808090;
  }

  .screen-container {
    display: grid;
    border: 2px solid #808090;
    background: #ffffff;
    border-radius: 5px;
    grid-template-columns: 1fr auto;
    grid-template-rows: 35px 1fr;
    overflow-x: hidden;
  }

  .screen-container > div:last-child {
    grid-column: 1 / -1;
  }

  .screen-container > div {
    display: flex;
    justify-content: flex-end;
    align-items: center;
  }

  .button-container {
    display: grid;
    grid-gap: 5px;
    grid-template-columns: 3fr 1fr;
  }

  /* .button-container > div {
    border: 1px solid blue;
  } */

  .action-buttons {
    display: grid;
    grid-gap: 5px;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(5, 1fr);
  }

  .arithmetic-buttons {
    display: grid;
    grid-gap: 5px;
    grid-template-rows: repeat(5, 1fr);
  }

  @media screen and (max-height: 780px) {
    main {
      margin-top: 0px;
    }
  }
</style>

<!-- 

  TODO:Add fonts for buttons and screen
  TODO:General sizing and responsiveness
  TODO:General colour and shape of calculator

  == NICE TO HAVES ==
  TODO:Animations for expression and value display
  TODO:Animations for screen outline if wronng syntax entered

  ==BUGS==
  TODO: Fix regex to allow minus numbers at start


 -->
<svelte:window on:keydown={handleKeyPresses} />

<div class="container">
  <main>
    <div bind:clientWidth={screenContainerWidth} class="screen-container">
      <div class="expression-screen">
        <div bind:clientWidth={expWidth}>
          <h4>
            {@html expressionWithEntities}
          </h4>
        </div>
      </div>
      <div class="equals-screen">
        {#if !!expressionEquals}
          <div id="EQUALS-WIDTH" bind:clientWidth={equalsWidth}>
            <h4>{equalsPressed ? `=` : `= ${roundedEquals}`}</h4>
          </div>
        {/if}
      </div>
      <div class="current-value-screen">
        <h1>{equalsPressed ? roundedEquals : value}</h1>
      </div>
    </div>
    <div class="button-container">

      <div class="action-buttons">
        <Button onClick={() => onNumericalKeyPress('1')}>1</Button>
        <Button onClick={() => onNumericalKeyPress('2')}>2</Button>
        <Button onClick={() => onNumericalKeyPress('3')}>3</Button>
        <Button onClick={() => onNumericalKeyPress('4')}>4</Button>
        <Button onClick={() => onNumericalKeyPress('5')}>5</Button>
        <Button onClick={() => onNumericalKeyPress('6')}>6</Button>
        <Button onClick={() => onNumericalKeyPress('7')}>7</Button>
        <Button onClick={() => onNumericalKeyPress('8')}>8</Button>
        <Button onClick={() => onNumericalKeyPress('9')}>9</Button>
        <Button onClick={() => onNumericalKeyPress('.')}>.</Button>
        <Button onClick={() => onNumericalKeyPress('0')}>0</Button>
        <Button type="action" onClick={() => onBackKeyPress()}>&crarr;</Button>
        <Button type="action" spanRow onClick={() => onClearKeyPress()}>
          CLEAR
        </Button>
      </div>
      <div class="arithmetic-buttons">
        <Button type="operator" onClick={() => onOperatorKeyPress('+')}>
          &plus;
        </Button>
        <Button type="operator" onClick={() => onOperatorKeyPress('-')}>
          &minus;
        </Button>
        <Button type="operator" onClick={() => onOperatorKeyPress('*')}>
          &times;
        </Button>
        <Button type="operator" onClick={() => onOperatorKeyPress('/')}>
          &divide;
        </Button>
        <Button type="operator" onClick={() => onEqualsPress()}>=</Button>
      </div>

    </div>
  </main>
</div>
