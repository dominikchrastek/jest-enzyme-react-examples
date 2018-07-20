# jest-enzyme-react-examples

[Jest](https://jestjs.io)
[Enzyme](http://airbnb.io/enzyme/)

## Examples

### Stateless component

#### Example component:

```js
// Button.tsx
// ...
const Button = ({ children, icon, onClick }) => (
  <button onClick={onClick}>
    {props.children}
    {icon && <Icon />}
  </button>
);
// ...
```

#### Example test:

There are 2 possible test scenarios, we can render Button or Button with icon, we have to test both
We have to check if `Button` was rendered properly (match snapshot), there are 2 scenarios (Button + Button with icon), so we have to write test for both of them

```js
// Button.spec.tsx
// ...
import { shallow } from 'enzyme';
import Button from '../Button';

describe('#Button', () => {
    it('should render', () => {
        const wrapper = shallow(<Button onClick={jest.fn()}>Text<Button>);
        expect(wrapper.getElement()).toMatchSnapshot();
    });

    it('should render with icon', () => {
        const wrapper = shallow(<Button onClick={jest.fn()} icon={SomeIcon}>Text<Button>);
        expect(wrapper.getElement()).toMatchSnapshot();
    });
});
```

---

### Component

#### Example component:

```js
// InputText.tsx
// ...
class InputText extends React.PureComponent {
  handleChange = ev => {
    this.props.onChange(ev.target.value);
  };

  render() {
    const { value, id } = this.props;

    return (
      <Input id={id} onChange={this.handleChange} value={value} type="text" />
    );
  }
}
// ...
```

#### Example test:

We have to check if `InputText` was rendered properly (match snapshot) + also test if the `handleChange` method works -> we have to check if the `onChange` callback was called with correct value

```js
// InputText.spec.tsx
// ...
import { shallow } from "enzyme";
import InputText from "../InputText";

describe("#InputText", () => {
  it("should render", () => {
    const wrapper = shallow(
      <InputText id="input" onChange={jest.fn()} value="value" />
    );
    expect(wrapper.getElement()).toMatchSnapshot();
  });

  it("should handleChange", () => {
    const onChange = jest.fn();
    const wrapper = shallow(
      <InputText id="input" onChange={onChange} value="value" />
    );

    const node = wrapper.instance();

    node.handleChange({ target: { value: "different value" } });
    expect(onChange).toBeCalledWith("different value");
  });
});
```

---

### Complex component

#### Example component:

```js
// Component.tsx
// ...
class Component extends React.PureComponent {
    state = {
        isOpen: true,
    }
    handleClose = () => {
        this.setState({isOpen: false})
        this.props.onClose()
    }
    handleClick = () => {
        this.handleClose()
        this.props.onClick()
    }

  render() {
    const { isOpen } = this.state;

    return (
      <>
        <Button onClick={this.handleClick}>Close</Button>
        {isOpen && (
            <SomeComponent />
        )}
      <>
    );
  }
}
// ...
```

#### Example test:

Here we have to check more things:

- Match snapshot for each possible state (there are 2 states - component isOpen and !isOpen)
- Test `handleClose` method - if state was set properly + callback was called
- Test `handleClick` method - if `handleClose` and callback (`onClick`) was called

```js
// Component.spec.tsx
// ...
import { shallow } from "enzyme";
import Component from "../Component";

describe("#Component", () => {
  it("should render open", () => {
    const wrapper = shallow(
      <Component onClose={jest.fn()} onClick={jest.fn()} />
    );
    expect(wrapper.getElement()).toMatchSnapshot();
  });

  it("should render closed", () => {
    const wrapper = shallow(
      <Component onClose={jest.fn()} onClick={jest.fn()} />
    );

    const node = wrapper.instance();

    node.setState({ isOpen: false });

    expect(wrapper.getElement()).toMatchSnapshot();
  });

  it("should handleClose", () => {
    const onClose = jest.fn();
    const wrapper = shallow(
      <Component onClose={onClose} onClick={jest.fn()} />
    );

    const node = wrapper.instance();

    node.handleClose();

    expect(node.state.isOpen).toBe(false);
    expect(onClose).toBeCalled();
  });

  it("should handleClick", () => {
    const onClick = jest.fn();
    const wrapper = shallow(
      <Component onClose={jest.fn()} onClick={onClick} />
    );
    const node = wrapper.instance();

    const handleClose = jest.spyOn(node, "handleClose");

    node.handleClick();
    expect(handleClose).toBeCalled();
    expect(onClick).toBeCalled();
  });
});
```

---

### HOC

#### Example component:

```js
// withUrl.tsx
// ...

function withUrl(Wrapped) {
    class Url extends React.PureComponent {
        static WrappedComponent = Wrapped;
        handleReset = () => {
            const url = resetUrl();
            this.props.history.push(url);
        };

        render() {
            return React.createElement(Wrapped, {
                ...this.props,
                urlParams: getUrlParams(), // wil return object of with url params
                onResetParams: this.handleReset,
            }
        }
    }
    return Url;
}
// ...
```

#### Example test:

Here we have to check more things:

- We have to match snapshots + checl if props wass passed to the child component
- We have to check if `handleReset` works (will not show it here, check some exmaple ⬆️)

```js
// withUrl.spec.tsx
// ...
import { shallow, mount } from "enzyme";
import withUrl from "../withUrl";

describe("#withUrl", () => {
  const Child = () => <div>Child</div>;
  const history = { push: jest.fn() };
  const Component = withUrl(Child);

  it("should render", () => {
    const wrapper = mount(<Component history={history} />);
    const childProps = wrapper.children(0).props();

    expect(typeof childProps.urlParams).toBe("object");
    expect(typeof childProps.onResetParams).toBe("function");
    expect(wrapper.getElement()).toMatchSnapshot();
  });
});
```

---

### Component with HOC

#### Example component:

```js
// Component.tsx
// ...
class Component extends React.PureComponent {
  handleReset = () => {
    const { value, onChange } = this.props;
    onResetParams(value);
  };

  render() {
    const {
      value,
      onResetParams // this is from HOC
    } = this.props;

    return (
      <>
        <button onClick={this.handleChange}>Reset</button>
      </>
    );
  }
}

export default URL(Component);
// ...
```

#### Example test:

- We have to use `WrappedComponent` here, then we will be able to pass props to the `Component` + `HOC`
- if we will have more than one HOC e.g. `URL(connect(mapStateToProps))`, then we have to do `ComponentWrapper.WrappedComponent.WrappedComponent` (for each hoc one WrappedComponent)

```js
// Component.spec.tsx

import { shallow } from "enzyme";
import ComponentWrapper from "../Component";

describe("#Component", () => {
  const Component = ComponentWrapper.WrappedComponent;
  const history = { push: jest.fn() };
  it("should render", () => {
    const wrapper = shallow(
      <ComponentWrapper
        history={history}
        onResetParams={jest.fn()}
        value="value"
      />
    );
    expect(wrapper.getElement()).toMatchSnapshot();
  });

  it("should handleReset", () => {
    const onResetParams = jest.fn();
    const wrapper = shallow(
      <ComponentWrapper
        history={history}
        onResetParams={onResetParams}
        value="value"
      />
    );

    const node = wrapper.instance();

    node.handleReset();
    expect(onResetParams).toBeCalledWith("value");
  });
});
```

---

### Some cases

#### Example:

- **don't** do this! it's anti pattern and it's hard to test, I see this really **often**
- also arrow function in render function is expensive (it's recreated on every render)

```js
// Component.tsx
const Component = ({ items, onClick }) => (
  <ul>
    {items.map(item => (
      <li key={item.id} onClick={() => onClick(item.id)}>
        {item.text}
      </li>
    ))}
  </ul>
);
```

#### Solution

- create two Components instead, it's easy to test + no arrow function

```js
// ComponentLi.tsx
class ComponentLi extends React.PureComponent {
  handleClick = () => {
    const { onClick, id } = this.props;
    onClick(id);
  };

  render() {
    return <li onClick={this.handleClick}>{this.props.children}</li>;
  }
}
// Component.tsx
const Component = ({ items, onClick }) => (
  <ul>
    {items.map(item => (
      <ComponentLi key={item.id} id={item.id} onClick={onClick}>
        {item.text}
      </ComponentLi>
    ))}
  </ul>
);
```
