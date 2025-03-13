# Technical Writing Assignment

For guidance on setting up and submitting this assignment, refer to the Marcy lab School Docs How-To guide for [Working with Short Response and Coding Assignments](https://marcylabschool.gitbook.io/marcy-lab-school-docs/fullstack-curriculum/how-tos/working-with-assignments#how-to-work-on-assignments).

## Prompt 1

In your own words, explain why React is a popular choice for building user interfaces. Make sure to mention at least one benefit, such as how it simplifies development, supports reusable components, or helps optimize performance. Feel free to include any specific features you find particularly helpful.

### Response 1

## Prompt 2

Explain how the useState hook is used in React to manage state within functional components. In your response, include an example of how useState might be used in a simple application and why managing state is important in building interactive user interfaces.

### Response 2

In order to give more control to rendering components, `useState` is one of many solutions that React came up with. What it does is that you can define variables using that same method which gives you access both to the value or reference itself in addition to a function that is meant to explicity dictate change that should accompany a re-render.

The simplest example of `useState`'s application would be a counter (with its corresponding 'update' button). It would look something like this:

```javascript
function App({}) {
  const [value, setValue] = useState(0);

  return (
    <div>
      <h1>{value}</h1>
      <button onClick={() => setValue(value+1)} />
    </div>
  )
}
```

If `useState` was not used, the value would still be updated, but it will not reflect on the render! This is because React has special hooks (`useState` included) which signal components when they should do so. This makes it so that the user can be guaranteed that they are getting the 'freshest' information available.

## Prompt 3

Describe the different ways the useEffect hook can be triggered in a React component. Include an explanation of how the dependency array influences its behavior. If possible, provide a code example for each scenario to illustrate your explanation.

### Response 3

## Prompt 4

The component below makes a mistake when using useEffect. When running this code, we will get an error from React! Please fix this code.

```js
const DogDisplay = () => {
  const [imgSrc, setImgSrc] = useState('https://images.dog.ceo/breeds/hound-english/n02089973_612.jpg');

  useEffect(async () => {
    try {
      const response = await fetch('https://dog.ceo/api/breeds/image/random');
      if (!response.ok) throw new Error(`Error: ${response.status}`)
      const data = await response.json();
      setImgSrc(data.message);
    } catch (error) {
      console.error(error);
    }
  }, []);

  return <img src={imgSrc} />
}
```

After fixing the code provide and explanation to what you fixed and why it needed to be fixed.

### Response 4

The very first error is that `useEffect` is only allowed to return a function. This is implicit with the syntax, since any callback `() => {}` returns an anonymous function by default (unless it has a `return` statement inside). `useEffect` from a functional standpoint should never be asynchronous itself to prevent race conditions where it is a possibility that the component unmounts before `useEffect` finishes executing. Putting the `async` keyword to any function makes it so that it returns not just a regular `Function`, but an `AsyncFunction`.

So let's fix that:

```js
  useEffect(() => { // made the callback synchronous by removing `async`
    const fetchImg = async() => { // created an asynchronous function inside that is called immediately
      try {
        const response = await fetch('https://dog.ceo/api/breeds/image/random');
        if (!response.ok) throw new Error(`Error: ${response.status}`)
        const data = await response.json();
        setImgSrc(data.message);
      } catch (error) {
        console.error(error);
      }
    }
    fetchImg(); // the immediate call
  }, []);
}
```

There's nothing really functionally incorrect after that, but one improvement could be that `imgSrc` could be null or blank at the beginning, and only have the fallback render in case there's an error with the fetches. This implies a conditional loading render that we will render until the fetch completes. That entails these changes:

```javascript
const App = () => {
  const [imgSrc, setImgSrc] = useState(null);
  const [isLoading, setIsLoading] = useState(null); // a new load state invoke a re-render whenever it changes

  useEffect(() => {
    const fetchImg = async() => {
      setIsLoading(true); // set loading to true while in the asynchronous function
      try {
        const response = await fetch('https://dog.ceo/api/breeds/image/random');
        if (!response.ok) throw new Error(`Error: ${response.status}`)
        const data = await response.json();
        setImgSrc(data.message);
      } catch (error) {
        console.error(error);
        setImgSrc('https://images.dog.ceo/breeds/hound-english/n02089973_612.jpg');
      } finally {
        setIsLoading(false); // finally, set isLoading to false to re-render the component
      }
    }
    fetchImg();
  }, []);

  return (
    <>
      {/* We conditionally render either a loading message or the img once it's ready */}
      { isLoading ? <h1>Loading...</h1> : <img src={imgSrc} /> }
    </>
  )
}
```
