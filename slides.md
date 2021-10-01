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
    Let's get started! <carbon:arrow-right class="inline"/>
  </span>
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
Points we will be touching. ---

State collocation again, just refers to the different levels we have to manage state and how can we use them. ---

UI vs Server State its just a topic about separating our UI or Client state from the Data that comes from APIs and that we usually also put in global state. ---

Typescript is just there for commodity, tools i will be tyalking about are bui;lt with it but does not mean this is a strong type architecture or best practices, i might dig on that later on.
 -->
---

<h1><span>State Collocation</span></h1>
<p  v-click>State management starts at component level.</p>
<p  v-click>React can manage state on its own... up close at least.</p>

<h3 v-click>Lifting State Up</h3>

<p  v-click>When several components need to reflect the same changing state, its recommend lifting the shared state up to their closest common parent.</p>

<v-click>
<p>Let's take a simple component for start:</p>

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

<p>Let's take a simple ThemeContext:</p>

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
<p v-click>❌ Requires more steps to test (this also applies to context at a smaller degree).</p>
<p v-click>❌ Global State can grew big.</p>

<p v-click>Some of these drawbacks have been addressed at some degree, Redux team it self recommends using Redux Toolkit and has a strongly recommended guideline.</p>

<!-- 
I will not go into too much details yet about redux since our next topic will visit it as well
-->

---

<h1><span>State Collocation</span></h1>
<p>Global State with Redux</p>

<p v-click>Let's take our Counter state and put it into Global State.</p>

<v-click>
<p>First we create our store and type definitions inferred from that store</p>

```ts{all|4|5-7|2|all}
// store.ts
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from '../features/counter/counterSlice';

export const store = configureStore({
  reducer: {
    counter: counterReducer,
  },
});

// Infer the `RootState` and `AppDispatch` types from the store itself
export type RootState = ReturnType<typeof store.getState>;
// Inferred type: {posts: PostsState, comments: CommentsState, users: UsersState}
export type AppDispatch = typeof store.dispatch;

```
</v-click>


<!-- 
This creates a Redux store, and also automatically configure the Redux DevTools extension so that you can inspect the store while developing.
-->

---

<h1><span>State Collocation</span></h1>
<p>Global State with Redux</p>

<p>We define our counterSlice which will handle all the logic for our counter feature.</p>

```tsx{all|1|4|6-14|9-13|16|17|all}
import { createSlice } from '@reduxjs/toolkit';

export interface CounterState { value: number; }
const initialState: CounterState = { value: 0, };

export const counterSlice = createSlice({
  name: 'counter',
  initialState,
  reducers: {
    increment: (state) => {
      state.value += 1; // RT uses immer unde the hood to allow "mutable" logic in reducers.
    },
  },
});

export const { increment } = counterSlice.actions; // Action creators are generated for each case reducer function
export default counterSlice.reducer;

```
<!-- 
Within a given feature folder, the Redux logic for that feature should be written as a single "slice" file, preferably using the Redux Toolkit createSlice API. (This is also known as the "ducks" pattern). 
-->
---

<h1><span>State Collocation</span></h1>
<p>Global State with Redux</p>

<p>We provide it with react-redux's provider.</p>

```tsx{all|1-2|6|all}
import { Provider as ReduxProvider } from 'react-redux';
import { store } from './app/store';

ReactDOM.render(
  <React.StrictMode>
    <ReduxProvider store={store}>
      <ThemeProvider>
        <App />
      </ThemeProvider>
    </ReduxProvider>
  </React.StrictMode>,
  document.getElementById('root')
);

```

<!-- 
-->

---

<h1><span>State Collocation</span></h1>
<p>Global State with Redux</p>

<p>And we get state from redux with useSelector:</p>

```tsx{all|1|5-6|10-12|all}
import { useSelector, useDispatch } from 'react-redux';

const Counter: FC = () => {
  // No longer lifted state, now Redux State
  const count = useSelector((state: RootState) => state.counter.value); // selective dependency for updates
  const dispatch = useDispatch();

  return (
    <Container maxWidth="sm">
      <CounterButton count={count} increment={() => dispatch(increment())} />
      <CounterDisplay count={count}>
        <CounterBanner count={count} />
      </CounterDisplay>
    </Container>
  );
};
```
<p v-click>Now every component in oour component tree can access the same way to Counter state (not that is needed).</p>

<!-- 
We refactored a bit this component to actually be the wrapper for the whole Counter feature. --- We might talk more about redux toolkit in the next topic so let's wrap it up
-->

---

<h1><span>State Collocation</span></h1>
<p>Let's summarize:</p>

<ul>
  <li v-click>State collocation is encouragement to organize state at different levels, not just Global State</li>
  <li v-click>Defining a ruleset about how deep we drill props or how up we lift state could help prevent further complications.</li>
  <li v-click>Identifying and separating components that handle business logic through state vs rendering components is also important!.</li>
  <li v-click>Defining a ruleset to identify when and how a component can be graduated to feature or smart component its crucial to maintain a large pool of components!.</li>
  <li v-click>Data that will not update often and needs to be provided to the whole app can go to Context.</li>
  <li v-click>State that will drive constant component updates and is needed wider in our app can go to Global State.</li>
</ul>

<!-- 
This is a simple and very just over the top (lacks the details) but more defined pattern to follow and break into specifics.
-->

---

<h1><span>UI State vs Server (Data) State</span></h1>
<p>A case for splitting our Global State:</p>

<p v-click>Web applications normally need to fetch data from a server in order to display it. They also usually need to make updates to that data, send those updates to the server, and keep the cached data on the client in sync with the data on the server.</p>

<p v-click>Over the last couple years, the React community has come to realize that "data fetching and caching" is really a different set of concerns than "State Management".</p>

<ul>
  <li v-click>Tracking our loading states in order provide better UX for loading times, this can be integrated with suspense in some cases.</li>
  <li v-click>Avoiding duplicate requests for the same dat, or again selectively updating pieces of it.</li>
  <li v-click>Behind the scene updates and re-fetch to sync data for different users.</li>
  <li v-click>Knowing when data is "out of date" to trigger this re-fetch</li>
  <li v-click>Caching... (one of the hardest things to do in programming).</li>
</ul>

<!-- 
Some of the concerns specific to server data state.
-->

---

<h1><span>UI State vs Server (Data) State</span></h1>
<p>A case for splitting our Global State:</p>

<p v-click>While we can use a state management library like Redux to fetch and cache data (thunk middleware), the use cases are different enough that it's worth using tools that are purpose-built for the data fetching use case.</p>

<p v-click>Over that same time some technologies have arised as pioneers of this pattern, we will discuss some of them:</p>

<ul>
  <li v-click>SWR (short for Stale While Revalidate)</li>
  <li v-click>React Query</li>
  <li v-click>RTK Query (short for Redux ToolKit Query)</li>
</ul>

<!-- 
-->

---

<h1><span>UI State vs Server (Data) State</span></h1>
<p>SWR and React Query</p>

<p>Both SWR and React Query have a similar approach to handling Server Data State, at least compared to RTK Query.</p>

<p v-click>Let's go over some points on them:</p>

<ul>
  <li v-click>Both provide a set of hooks as their api to execute fetching, mutating, cahcing and cache invalidation.</li>
  <li v-click>Their role is clear. To remove asynchronous data management and boilerplate from your application and replace it with just a few lines of code.</li>
  <li v-click>At the same time this abstraction can make things a bit obscure compared to plain Redux</li>
  <li v-click>At the time they are not framework agnostic, SWR integrates well and supports many features of Next.js, react query is a bit more universal.</li>
  <li v-click>Both also have experimental support for Suspense.</li>
  <li v-click>Both are built with TypeScript so they provide out of the box implementation with it if needed.</li>
</ul>

<!-- 
-->

---

<h1><span>UI State vs Server (Data) State</span></h1>
<p>SWR and React Query</p>

<p v-click>Let's take a quick example for implementing SWR</p>

<v-click>

<p>Configuring SWR globally:</p>

```tsx{all|1-6|all}
<SWRConfig
  value={{
    suspense: true,
    fetcher: (url) => axios.get(url).then((res) => res.data),
  }}
>
  <ThemeProvider>
    <App />
  </ThemeProvider>
</SWRConfig>
```
</v-click>
<!-- 
-->

---

<h1><span>UI State vs Server (Data) State</span></h1>
<p>SWR and React Query</p>

<p>Using it directly:</p>

```tsx{all|5|9-11|all}
import useSWR from 'swr';

const PostList: FC = () => {
  /* error and loading state can also be obtained from useSWR */
  const { data } = useSWR('https://jsonplaceholder.typicode.com/posts');

  return (
    <ul>
      {data?.map((post: Post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
};

```
<p v-click>Notice how we dont need to validate data due to experimental Suspense support.</p>
<!-- 
-->

---

<h1><span>UI State vs Server (Data) State</span></h1>
<p>SWR and React Query</p>

<p>When building a web app, you might need to reuse the data in many places of the UI. It is incredibly easy to create reusable data hooks on top of SWR:</p>

```tsx
const usePosts = () => {
  const { data, error } = useSWR(`https://jsonplaceholder.typicode.com/posts`)

  return {
    posts: data,
    isLoading: !error && !data,
    isError: error
  }
};

```
<!-- 
We would end up consuming usePosts wherever we need posts data.
-->

---

<h1><span>UI State vs Server (Data) State</span></h1>
<p>SWR and React Query</p>

<p>This is how we would wrap with Suspense:</p>

```tsx{all|4|5|6|all}
const PostPageSWR: FC = () => {
  return (
    <Box sx={{ display: 'flex' }}>
      <ErrorBoundary fallback={<p>Could not fetch data</p>}>
        <Suspense fallback={<CircularProgress />}>
          <PostListSWR />
        </Suspense>
      </ErrorBoundary>
    </Box>
  );
};

```

<p v-click>As you can see is easy to use and share, it handles all of the data in a global cache and is also lightweight. React Query works in a very similar way.</p>
<!-- 
ErrorBoundary catches errors bubling up from components below.
-->

---

<h1><span>UI State vs Server (Data) State</span></h1>
<p>RKT Query</p>

<p>On the other hand RKT Query builds on top oc its own RTK API:</p>

```ts{all|1|3-8|all}
// Need to use the React-specific entry point to import createApi
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

export type Post = {
  userId: number;
  id: number;
  title: string;
  body: string;
};

```
<!-- 
-->

---

<h1><span>UI State vs Server (Data) State</span></h1>
<p>RKT Query</p>

```ts{all|2|4-6|7-14|all}
// Define a service using a base URL and expected endpoints
export const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({
    baseUrl: 'https://jsonplaceholder.typicode.com',
  }),
  endpoints: (builder) => ({
    getPosts: builder.query<Post[], null>({
      query: () => `posts`,
    }),
    getPostById: builder.query<Post, number>({
      query: (id) => `post/${id}`,
    }),
  }),
});

// auto-generated based on the defined endpoints
export const { useGetPostsQuery, useGetPostByIdQuery } = api;
```
<!-- 
-->

---