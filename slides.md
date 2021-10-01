---
# try also 'default' to start simple
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: monaco
monaco: true
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
---

# State Management

A look into state management in react apps

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Lets get started! <carbon:arrow-right class="inline"/>
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" alt="GitHub"
    class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>
<!-- 
Initially this was a presentation for State and patterns topic for Lib 2 and still is, just expands a bit beyond the Redux vs Context for state management topic.
-->
---

<h1><span>Quick Refresher on State</span>🥤</h1> 

<p  v-click>In simple terms, State is a JavaScript object that represents the current situation of a component, a specific feature or an entire application and is usually changed based on user interaction with the app.</p>
<p v-click>State Managment starts at component level, our components react to their own state changes, but using component state alone can be painful 🤕 as we might need to know about those changes in other components.</p>
<p v-click>At some degree is posible to manage state between components by lifting it up (will explain later). When dependent components are many through the app, lifting is no longer a good choice.</p>
<h2 v-click class="text-center">Enter Global State 🌎</h2>
<p v-click>It is at this point that we usually move state up, outside of components into a global / application state.</p>
<p v-click>✔️ Global can still be organized by features.</p>
<p v-click>✔️ Components have a single source of thruth, no need to sync.</p>
<p v-click>✔️ Every component can have easy access to it.</p>


<!--
Changes in the application happen usually at component level.

Before global state, some other solutions may exist but they rely more on component composition than actual state management (smart vs dumb components)

-->

---
layout: image-right
image: https://source.unsplash.com/collection/94734566/1920x1080
---

<h1><span>So, global it is then</span>😎</h1>

<p  v-click>Not quite...</p>
<br>

<h3 v-click>State Collocation</h3>
<ul>
  <li v-click>Lifting State when maintainable</li>
  <li v-click>Context specific cases</li>
  <li v-click>Global State for larger scale</li>
</ul>
<br>
<h3 v-click>UI State vs Server Data State</h3>
<ul>
  <li v-click>A case for splitting</li>
  <li v-click>Tools for spliting</li>
  <li v-click>Best practices and implementations</li>
</ul>

<span  v-click class="text-gray-500">The goal is a solid set of State Management rules / practices</span>
<br>
<span  v-click class="text-gray-500"> ...dont mind the TypeScript 🙀 (or mind it if you want...)</span>

<!--
Points we will be touching.

State collocation again, just refers to the different levels we have to manage state and how can we use them.

UI vs Server State its just a topic about separating our UI or Client state from the Data that comes from APIs and that we usually also put in global state.
 -->
---

<h1><span>State Collocation</span></h1>
<p  v-click>State management starts at component level.</p>
<p  v-click>React can manage state on its own... up close at least.</p>

<h3 v-click>Lifting State Up</h3>

<p  v-click>When several components need to reflect the same changing state, its recommend lifting the shared state up to their closest common parent.</p>

<v-click>
<p>Lets take a simple component for start:</p>

```tsx {all|2|3|5|all}
const Counter: FC = () => {
  const [count, setCount] = useState(0);
  const increment = () => setCount((count) => count + 1);

  return <button onClick={increment}>{count}</button>;
};
```
</v-click>

<p v-after>Counter component handles its own state for incrementing count.</p>

<!-- 
Think of this step as just a first line of deffence agaisnt sate managment problems.
-->
---

<h1><span>State Collocation</span></h1>
<p>State management starts at component level.</p>

<p>What happens if we need that count state somewhere else.</p>
<v-click>
<p>Like in the a CounterDisplay:</p>

```tsx {all|5|6|all}
const CounterPage: FC = () => {
  // Where is the count?
  return (
    <Container maxWidth="sm">
      <Counter /> {/* i have the count */}
      <CounterDisplay /> {/* i need count as well*/}
    </Container>
  );
};
```
</v-click>

<p v-after>CounterDisplay needs the count which is cointained in the counter... no luck there 😢.</p>

<!-- 
Does this mean we need to go global? NO!
-->

---

<h1><span>State Collocation</span></h1>
<p>State management starts at component level.</p>

```tsx {all|3|4|8|9|15|16|20|21|all}
// Common parent CounterPage
const CounterPage: FC = () => {
  const [count, setCount] = useState(0);
  const increment = () => setCount((count) => count + 1);

  return (
    <Container maxWidth="xl">
      <Counter count={count} increment={increment} />
      <CounterDisplay count={count} />
    </Container>
  );
};

// Counter
const Counter: FC<CounterType> = ({ count, increment }) => {
  return <button onClick={increment}>{count}</button>;
};

// CounterDisplay
const CounterDisplay: FC<CounterDisplayType> = ({ count }) => {
  return <p>Current count is {count}</p>;
};
```

<!-- 
So we lift our state to CounterPage, the common parent and pass the state as props down 1 level.
-->
---

<h1><span>State Collocation</span></h1>
<p>State management starts at component level.</p>

<p>Lifting state up is not a bad solution for very specific components coupled together.</p>

<p v-click>This is like just the first line of defense against State Managment problems, its worth considering when deppendent components are "close" to each other in our component tree, therefore, prop drilling can be solved by composition.</p>

<v-click>
<p>Consider an example where CountDisplay needs to pass count to its own children:</p>

```tsx {all|1|5|all}
const CounterDisplay: FC<CounterDisplayType> = ({ count }) => {
  return (
    <>
      <p>Current count is {count}</p>
      <CounterBanner count={count} /> {/* drilling second lvl */}
    </>
  );
};
```
</v-click>

---

<h1><span>State Collocation</span></h1>
<p>State management starts at component level.</p>

<p>Our CounterBanner component will get that count from its parent:</p>

```tsx {all|1|2|4|5|all}
const CounterBanner: FC<CounterBannerType> = ({ count }) => {
  const bannerColor = count >= 10 ? 'lightgreen' : 'pink';
  const countMessage =
    count >= 10
      ? `Congrats! you reached ${count} in the count`
      : `Keep it up, you're still under 10`;

  return (
    <div style={{ background: bannerColor, padding: '4px 18px' }}>
      <h3>{countMessage}</h3>
    </div>
  );
};
```

---

<h1><span>State Collocation</span></h1>
<p>State management starts at component level.</p>

<p>We can eliminate the drilling by refactoring a bit to allow children components:</p>

```tsx {all|2|6|all}
// CounterDisplay
const CounterDisplay: FC<CounterDisplayType> = ({ count, children }) => {
  return (
    <>
      <p>Current count is {count}</p>
      {children}
    </>
  );
};
```

---

<h1><span>State Collocation</span></h1>
<p>State management starts at component level.</p>

<p>And then compose that original component by passing CounterBanner as child component:</p>

```tsx{all|10|all}
// CounterPage (Parent)
const CounterPage: FC = () => {
  const [count, setCount] = useState(0);
  const increment = () => setCount((count) => count + 1);

  return (
    <Container maxWidth="xl">
      <Counter count={count} increment={increment} />
      <CounterDisplay count={count}>
        <CounterBanner count={count} />
      </CounterDisplay>
    </Container>
  );
};
```
<p v-click>This requires no changes for CounterBanner and count prop is not being drilled more than 1 level down.</p>
<p v-click>This is also an example on how State Collocation just made children components dumb, they only handle rendering, while state logic is in parent.</p>
<!-- 
This is just a tip to maybe save 1 or 2 levels of prop drilling when lifting up state makes sense. --- 
And again, setting a rule like how deep we drill or how high we lift is something that needs to be done, maybe not right now but when preparring our architecture.
-->

---

<h1><span>State Collocation</span></h1>
<p>Global State or Context</p>

<p>When we found that components further in the component tree need that state, we will end up either lifting too high or drilling props too deep, this is when moving that state to a Global State is a better option.</p>

<p v-click>What about React's own Context api?</p>

<h3 v-click>Context</h3>
<ul>
  <li v-click>Is natively in react</li>
  <li v-click>Easy to setup, provide and consume.</li>
  <li v-click>Context is meant to provide global data not prompt to change often.</li>
  <li v-click>Is not meant to manage selective updates and re-rendering</li>
</ul>

<p v-click>Context CAN be turned into a State Management tool, but just because it can, does not mean it should (or that is the best solution, which is not).</p>

<p v-click>Context is a very good option to provide data that doesn't update often and that is needed throughout the whole application.</p>

<!-- 
This means components further in the component tree need that state, that is when the app requires lifting to high or / and drilling too deep.
-->

---

<h1><span>State Collocation</span></h1>
<p>Context</p>

<p>Lets take a simple ThemeContext:</p>

```tsx{all|3-5|6|all}
// Could also come from API
const defaultTheme = {
  accentColor: 'blue',
  textColor: 'black',
  bgColor: 'lightgrey',
  changeTextColor: (color: string) => {}, // this will be set by provider
};

```
<p v-click>This could be considered initial state for theme.</p>

<!-- 

-->

---

<h1><span>State Collocation</span></h1>
<p>Context</p>

<p>Creating Context and Provider:</p>

```tsx{all|1|3|4|6-12|14-18|all}
export const ThemeContext = createContext<ThemeContextInterface>(defaultTheme);

export const ThemeProvider: FC = ({ children }) => {
  const [theme, setTheme] = useState<ThemeContextInterface>(defaultTheme);

  const changeTextColor = (color: string) => {
    if (theme.textColor !== color) {
      const updatedTheme = { ...theme };
      updatedTheme.textColor = color;
      setTheme(updatedTheme);
    }
  };

  return (
    <ThemeContext.Provider value={{ ...theme, changeTextColor }}>
      {children}
    </ThemeContext.Provider>
  );
};

```

<!-- 
Create the context --- Create the provider --- Create local state --- Modify the state -- Return the provider
-->

---

<h1><span>State Collocation</span></h1>
<p>Context</p>

<p>Providing the Context:</p>

```tsx{all|2-4|all}
  <React.StrictMode>
    <ThemeProvider>
      <App />
    </ThemeProvider>
  </React.StrictMode>,

```
<v-click>
<p>Quick note on porvider's order:</p>

```tsx{all|2|3|4|all}
  <React.StrictMode>
    <ContextProvider_1>
      <ThemeProvider>
        <ContextProvider_2>
          <App />
        </ContextProvider_2>
      </ThemeProvider>
    </ContextProvider_1>
  </React.StrictMode>,

```
</v-click>

<!-- 
Create the context --- Create the provider --- Create local state --- Modify the state -- Return the provider
-->

---

<h1><span>State Collocation</span></h1>
<p>Context</p>

<p>Consuming the context:</p>

```tsx{all|2|8|11|4|all}
const CounterDisplay: FC<CounterDisplayType> = ({ count, children }) => {
  const { textColor, changeTextColor } = useContext(ThemeContext);

  console.log('display rendering...'); // component will rerender when color is changed

  return (
    <>
      <span style={{ margin: '0 12px', color: textColor }}>
        Current count is {count}
      </span>
      <Button variant="contained" onClick={() => changeTextColor('red')}>
        Change text color!
      </Button>
      {children}
    </>
  );
};

```

<!-- 
We get textColor and changeTextColor from context and use them in component, re-render will occur when button is clicked.
-->

---

<h1><span>State Collocation</span></h1>
<p>Context</p>

```tsx{all|2|17|9|all}
const CounterBanner: FC<CounterBannerType> = ({ count }) => {
  const { accentColor } = useContext(ThemeContext);
  const bannerColor = count >= 10 ? 'lightgreen' : 'pink';
  const countMessage =
    count >= 10
      ? `Congrats! you reached ${count} in the count`
      : `Keep it up, you're still under 10`;

  console.log('banner rendering...'); // Why?

  return (
    <div
      style={{
        background: bannerColor,
        padding: '4px 18px',
        marginTop: '12px',
        color: accentColor, // this is not changed in context but still re-rendering
      }}
    >
      <h3>{countMessage}</h3>
    </div>
  );
};

```

<!-- 
Context provided that theme state and a way to update it, however we now have an unexpected re-rendering in CounterBanner. So Context can manage state at some degree but it has a serious drawback in its unselective updates and re-renderings. This might not cause a problem when the data we store in them is not updated frequently and is needed through the whole application, like a full sized Theme.
-->

---

<h1><span>State Collocation</span></h1>
<p>Global State with Redux</p>

<p v-click>State that will update frequently, that will drive selective re-rendering and that is needed wider in our application, sits better at our Global State</p>

<h3 v-click>Redux</h3>
<p v-click>✔️ Wideley used and known.</p>
<p v-click>✔️ Framework agnostic (Mostly).</p>
<p v-click>✔️ Flexible workflow that allows more granular control.</p>
<p v-click>✔️ Dev tools are pretty cool.</p>

<p v-click>Redux has been out there for a while, its well documented and can work with many types of applications.</p>

<!-- 
It also has some drawbacks as it is a big tool with powerful features.
-->

---

<h1><span>State Collocation</span></h1>
<p>Global State with Redux</p>

<h3>Redux</h3>
<p v-click>❌ Boilerplate can be descriptive but also daunting.</p>
<p v-click>❌ Tricky to maintain Global State and Context or component state together.</p>
<p v-click>❌ Requires more steps to test (this also applys to context at a smaller degree).</p>
<p v-click>❌ Global State can grew big.</p>

<p v-click>Some of these drawbacks have been addressed at some degree, Redux team it self recommends using Redux Toolkit.</p>

<p v-click>Redux Toolkit reduces boilerplate and adds sugar syntax (or should i say "mutable") through immer.</p>

<!-- 
I will not go into details yet about redux since our next topic will visit 
-->

---

<h1><span>State Collocation</span></h1>
<p>Let's summarize:</p>

<ul>
  <li v-click>State collocation is encouragement to organize state at different levels, not just Global State</li>
  <li v-click>Defining ruleset about how deep we drill props or how up we lift state could help prevent further complications.</li>
  <li v-click>Identifying and separating components that handle business logic through state vs rendering components is also important!.</li>
  <li v-click>Data that will not update often and needs to be provided to the whole app can go to Context.</li>
  <li v-click>State that will drive constant component updates and is needed wider in our app can go to Global State.</li>
</ul>

<!-- 
-->

---

<h1><span>UI State vs Server (Data) State</span></h1>
<p>A case for splitting our Global State:</p>

<ul>
  <li v-click>State collocation is encouragement to organize state at different levels, not just Global State</li>
  <li v-click>Defining ruleset about how deep we drill props or how up we lift state could help prevent further complications.</li>
  <li v-click>Identifying and separating components that handle business logic through state vs rendering components is also important!.</li>
  <li v-click>Data that will not update often and needs to be provided to the whole app can go to Context.</li>
  <li v-click>State that will drive constant component updates and is needed wider in our app can go to Global State.</li>
</ul>