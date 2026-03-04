# forwardRef accepts a render function as an argument. React calls this function with props and ref:
```js
const MyInput = forwardRef(function MyInput(props, ref) {
  return (
    <label>
      {props.label}
      <input ref={ref} />
    </label>
  );
});
```