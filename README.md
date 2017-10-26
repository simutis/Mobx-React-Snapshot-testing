### In this tutorial we will cover the following SNAPSHOT testing scenarios:

&nbsp;

**1. Presentational/functional component snapshot**<br/>
**2. Container component snapshot**<br/>
**- 2.1 mobx-react/inject problem**<br/>
**- 2.2 SHALLOW rendering - "ENZYME/shallow" vs "react-test-rendered/shallow**<br/>
**- 2.3 Full render with 'mobx-react/inject' mock**<br/>
**3. React-Intl problem**

&nbsp;&nbsp;

### 1. Presentational/functional component snapshot
As presentational components usually are the simplest ones, we just use 'react-test-renderer' to generate snapshot:

```javascript
import React from 'react';
import renderer from 'react-test-renderer';
....
  it('renders correctly', () => {
    const segmentBarChart = renderer.create(<SegmentBarChart {...mockData} />).toJSON();
    expect(segmentBarChart).toMatchSnapshot();
  });
});
```

### 2. Container component snapshot
Container components snapshots are more complicated to render because:
- usually they have child components, that might fail if they missing some properties, as a result you should mock and pass the data in order to get them rendered;
- container itself might be decorated with "mobx-react/inject";
- child components might be decorated with 'mobx-react/inject";
&nbsp;
#### 2.1 mobx-react/inject problem
Because some of the componenst properties might be injected via 'mobx-react/inject' it might happen that the component will fail to render because of the missing properties.
The most simple solution would be to pass the properties to component directly.
&nbsp;
Another option: export the CLASS or function definition in the module without inject, so you can access the class directly:
```javascript
export const someComponent = (props) => <div ......./>

export default inject('uiStore')(someComponent);
```

> PROBLEM: is that you will not be able to pass the properties directly to a child components if they are wrapped with "mobx-react/inject".
In this case we have 2 options here:
- mock 'mobx-react/inject' ourselves
- use shallow renderer that will not execute child components
&nbsp;
####  2.2 SHALLOW container rendering - “ENZYME/shallow” vs "react-test-rendered/shallow"
The problem is that "ENZYME/shallow" tries to execute all child components, even if it renders only the top level
> So we should use "react-test-rendered/shallow" all the time as it does not have the mentioned problem, as a result your test will not be dependent on the failure of child component.
```javascript
import ReactShallowRenderer from 'react-test-renderer/shallow';

// importing class only without inject
import { SomeComponent } from './SomeComponent';
const renderer = ReactShallowRenderer.createRenderer();

it('should renders correctly', () => {
  const SomeComponent = renderer.render(
    <SomeComponent {...mockProps} />
  );

  expect(passionsTableContainer).toMatchSnapshot();
});
```
&nbsp;
#### 2.3 Full container render with 'mobx-react/inject' mock
In some cases if we want to have full container render, we can mock 'mobx-react/inject"

file: \__mocks__/mobx-react.js
```javascript
import React, { Component } from 'react';

let storesMock = {};

export function observer(component) { return component; };

function createStoreInjector(component) {
  class Injector extends Component {
    render() {
      const newProps = {};
      for (const key in this.props) if (this.props.hasOwnProperty(key)) {
        newProps[key] = this.props[key];
      }
      Object.assign(newProps, storesMock);
      return React.createElement(component, newProps);
    }
  }
  return Injector;
};

export function inject() {
  return function injectWrapper (componentClass) {
    return createStoreInjector(componentClass);
  };
};

export function injectStoreMocks(mocks) {
  Object.assign(storesMock, mocks);
};

export function cleanMocks() {
  storesMock = {};
};
```
### Example:
&nbsp;
```javascript
jest.mock('mobx-react');
// 'injectStoreMocks' is defined in our helper <root>/__mock__/mobx-react.js
// after we mock it we have additional methods to set and clean mocks
import { injectStoreMocks, cleanMocks } from 'mobx-react';

import React from 'react';
import renderer from 'react-test-renderer';
import PersonaCard from './PersonaCard';

const storeMock = {
  uiPersona: {..}
  uiStore: {..}
};

describe('PERSONAS CONTAINER', () => {
  beforeAll(() => {
    injectStoreMocks(storeMock);
  });

  afterAll(() => {
    cleanMocks();
  });

  it('renders correctly', () => {
    const personaCard = renderer.create(
      <PersonaCard />
    ).toJSON();
    expect(personaCard).toMatchSnapshot();

```
&nbsp;
### 3. React-intl problem

If your component or its children has 'react-inlt' as dependency, it will fail to render and the error will be thrown.
The solution is to use a separate helper for that:
https://github.com/yahoo/react-intl/wiki/Testing-with-React-Intl
we have defined it in our testing helpers folder

```javascript
import createComponentWithIntl from 'createComponentWithIntl';
import someComp from './someComp';

it('should render correctly', () => {
const mockProps = {....};

const someComp = createComponentWithIntl( <someComp {...mockProps} />).toJSON();

expect(someComp).toMatchSnapshot();


```






