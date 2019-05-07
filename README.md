# Dashboard
1. **Component architecture**
    The structure would be based on the parent component which is the dashboard and its relationship with the rest of the components. The image below shows the breakdown of the components on a high level and how they relate with each other. 

    Components such as the Done/Next buttons (Dialog Box) are rendered depending on certain conditions and certain choices the user makes. I have attached some of the component codes to show how I would design the code flow of the application. Note: Some of the codes were ommited in order to focus on the logical picture

    ###### Main Dashboard Component
```
import React, { Component } from 'react';
import Dialog from './Dialog'
import InvoiceItemsList from './InvoiceItemsList'

class Dashboard extends React.Component {

  componentDidMount() {
    //actions to get invoice list here
    return (
      <div><Dialog className="dialog-style" /></div>
    )
  }
  handleButtonClick = () => {
    // handle dialog pop up here
  }
  handleDelete = () => {
    //triggers delete invoice action
  }
  handleEdit = () => {
    //triggers edit invoice action
    //logic to prompt the dialog box component 
    return (
      <div>
        <Dialog
          className="dialog-style"
          invoiceItem={this.props.invoiceItem}
        />
      </div>
    )
  }
    componentWillUnmount() {
    //unmounts the component
    this.setState({});
  }
  render() {
    return (
      <div className="">
        <div>
          <h3 className="invoice-list-header">Invoice list:</h3>
          <button
            className="add-new-invoice-btn"
            onClick={this.handleButtonClick}
          >
            Add new Invoice
          </button>
        </div>
        <div
          className="invoice-list">
          <InvoiceItemsList
            Ondelete={this.handleDelete}
            onEdit={this.handleEdit} />
        </div>
      </div>
    )
  }
}

```
The dialog component represented is for thee web version, implementation of the mobile version would be in the mobile section

   ###### Dialog Component
```
import React from 'react'
import Toggle from './Toggle' // code for toggle component incomplete, pseudo code here
import Payment from './PaymentDashboard' //code for payment dashboard

class Dialog extends React.Component {

  constructor(props) {
    super(props)
    this.state = {
      isToggleOn: false,
      fadind: true
    }
  }
  componentDidmount() {
    //calls the action related to populating an existing invoice if it exists
  }
  handleToggle = () => {
    if (!this.state.isToggleOn) {
      this.setState({ isToggleOn: true })

    }
  }

  handleSubmit = () => {
    //trigger action that submits entry for new invoice 
    //trigger function to close modal
  }

  handleNext = () => {
    return (
      <div>
        <PaymentDashboard />
      </div>
    )
  }
  componentWillUnmount() {
    //unmounts the component
    this.setState({});
  }


  render() {
     {/*Fields will be automatically 
    updated if the feature is used to edit an invoice>*/}
    
    if (!isToggleOn && onInvoiceInfoTab) {
      return (
        <div className="dashboard-wrapper">
          <div className="column1">
            <p>Invoice information</p>
            <p
              className={`${this.state.fading ? 'faded' : ''}`}>
              Payments
            </p>
          </div>
          <div>
            <p>Date:</p>
            <Datepicker value={this.state.invoiceItem.date} />
            <p>Subject:</p>
            <input value={this.state.invoiceItem.subject}></input>
          </div>

          <div>
            <p>Retrieve amount from bank account</p>
            <Toggle
              onClick={this.handleToggle}
            />
          </div>

          <div>
            <button
              onClick={this.handleSubmit}>
              Done
            </button>
          </div>
        </div>
      )
    }
    else {
      return (
        <div className="dashboard-wrapper">
          <div className="column1">
            <p
              className={`${this.state.fading ? 'faded' : ''}`}>
              Invoice information
            </p>
            <p>Payments</p>
          </div>
          <div>
            <Datepicker value={this.state.invoiceItem.date} />
            <Datepicker />
            <p>Subject:</p>
            <input value={this.state.invoiceItem.subject}></input>
          </div>

          <div>
            <p>Retrieve amount from bank account</p>
            <Toggle onClick={this.handleToggle} />
          </div>
          <div><button onClick={this.handleNext}>Next</button></div>
        </div>
      )
    }
  }
}

export default Dialog
```
Conditional rendering was used in the code above in order to swith between the Invoice Information Dialog and Payment Dialog
    
The code below shows the Payment Dialog Component

###### Payment Dialog Component

```
import React from 'react';
//code not written for this component 
import SearchBox from './SearchBox' 
//code not written for this component 
import BankTransfersList from 'BankTransfersList' 


class PaymentDashboard extends React.Component {

  constructor(props) {
    super(props)
    this.state = {
      searching: false,
      selectedTransfer: {}
    }
  }

  componentDidMount() {
    //trigger actions needed for this component when initially rendered
  }
  handleSearch = (event) => {
    //trigger action needed to send the information from the search component to the backend
    this.setState({ searching: event.target.value.length > 0 });
  }
  handleSelectedTransfer = (event) => {
    this.setState({ selectedTransfer: event.target.value })
  }

  render() {
    return (
      <div>
        {!this.state.searching ?
          <div>
            <SearchBox
              onChange={this.handleSearch} />
          </div> :
          <div>
            <SearchBox
              onChange={this.handleSearch} />
            <BankTransfersList
              onSelect={this.handleSelectedTransfer} />
          </div>}
        <button
          type="button"
          disabled={!this.state.selectedTransfer}>
          Done
       </button>
      </div>

    )
  }
}
```

2. **State management**
    The store is used to store the state of the application and prevent direct mutation. An action needs to be triggered for the store to update the state of the application. Below are snippets of some of the actions and how the state flows through to the component

    Using Redux to manage the state of the application we would create action creators to trigger specific actions, reducers to handle data formating and management and one central store to store the entire state of the application
    
    Firstly we wrap the app with the Provider in order to subscribe the app to the store, this is performed at the root javascript file of the application
```
ReactDOM.render(
  <Provider store={store}>
  <App/>
  </Provider>,
  document.getElementById('root')
);
```
Create and initialise redux store
```
import { createStore, applyMiddleware } from 'redux';
import ReduxThunk from 'redux-thunk';
import reducers from './reducers/index';

const store = createStore(reducers, {}, applyMiddleware(ReduxThunk));
export default store;
```
Pseudo code for combining reducers, this would usually be used in a large system with multiple reducers. If just one reducer is present or a small application, there would be no need for `combineReducers`
```
import { combineReducers } from 'redux';
import invoiceReducer from './invoiceReducers'
export default combineReducers({
  invoice: invoiceReducer
});

```
Actions pseudocode for creating and editing invoice

```
const ADD_INVOICE_SUCCESS = 'add_invoice_success'
const EDIT_INVOICE_SUCCESS = 'edit_invoice_success'
const ADD_INVOICE_FAILURE = 'add_invoice_failure'
const EDIT_INVOICE_FAILURE = 'edit_invoice_failure'

//import axios here

const addInvoice = (dispatch, invoice) => {
  dispatch({
    type: ADD_INVOICE_SUCCESS,
    payload: { invoice }
  });
}

const editInvoice = (dispatch, invoice) => {
  dispatch({
    type: EDIT_INVOICE_SUCCESS,
    payload: {invoice}
  })
}

export function createInvoice(date, subject, amount, iban) {
  //usually the authorization headers are set up here
  //axios would be used to make an ajax call to the endpoint
  return dispatch => axios.post('/api/v1/invoice', date, subject, amount, iban)
    .then((response) => {
      dispatch(addInvoice(response.data));
    })
    .catch((error) => {
      dispatch({ type: ADD_INVOICE_FAILURE, error: error.response.data });
    });
}

export function updateInvoice(id,date, subject, amount) {
  //usually the authorization headers are set up here
  //axios would be used to make an ajax call to the endpoint
  return dispatch => axios.put(`/api/v1/invoice/${id}`, id,date, subject, amount)
    .then((response) => {
      dispatch(updateInvoice(response.data));
    })
    .catch((error) => {
      dispatch({ type: EDIT_INVOICE_FAILURE, error: error.response.data });
    });
}
```
 Invoice Reducer  
```
import { ADD_INVOICE_SUCCESS } from '../actions/invoiceActions'
const INITIAL_STATE = {
  invoice: {
    date: '',
    subject: '',
    amount: 0,
    iban: ''
  }
}

export default (state = INITIAL_STATE, action) => {
  switch (action.type) {
    case ADD_INVOICE_SUCCESS:
      return {
        ...state,
        invoice: action.invoice
      }
      case EDIT_INVOICE_SUCCESS:
      return {
        //return the new state once an invoice is updated
      }
    default:
      return state;
  }
}
```
 ###### Dialog box Component
Connecting the main component to the store and actions 
Import block
```
//connecting Redux
import connect from 'react-redux'
import {createInvoice} from '../actions/invoiceActions.js' //import the createinvoice action creators
```
//in the case of editing an existing invoice, the invoice is automatically loaded to the component using the lifecycle method componentDidMount()

```
  componentDidmount() {
    //calls the action related to populating an existing invoice if it exists
    this.props.actions.createInvoice({date, subject, amount, iban});
  }
```
This is to listen to any changes made to the state  of the application, if any, the component will re-render
```
  static getDerivedStateFromProps(nextProps, prevState) {
  if (nextProps.invoice !== prevState.invoice) {
    return ({ total: nextProps.invoice }) // <- this is setState equivalent
  }
}
```

The final block connects the component to the store to receive updated states
```
 const mapStateToProps = ({invoice}) => {
  return {
    invoice: state.invoice
  };
}
export default connect(mapStateToProps, mapDispatchToProps)(Dialog);
```
3. **Responsiveness**
    Both Mobile and Web devices share similar logic when it comes to the dialog box, the difference is in the component structure. The solution is to seperate the logic bit and cause a conditional render depending on the size of the screen used to view the application

```
componentDidMount() {
    window.addEventListener("resize", this.resize.bind(this));
    this.resize();
}
resize() {
    if( window.innerWidth <= 760)
    this.setState({isMobile: true});
}

render() {
    if(isMobile){
        return (
        //mobile display)
    }
    else{
        //return web display
    }
}
```

4. **Test Strategy**
    To effectively test this feature all the parts of the redux flow needs to be tested
    Redux documentation recommends the use of jest and enzyme as it runs on the node environment and would not require accessing the DOM
    
    To test the actions and reducer functions, the store would need to be mocked inorder to simulate the redux flow
    
    Another test that can be run to crosscheck to ease of the use of the app from the user's perspective is the e2e test ( end to end testing). It runs by simulating the user's movement through the app. Nightwatch.js is an easy package to use and has clear documentations on how to run tests for the front end.
    
5. **User Experience and Design**
Some shortcomings I noticed with the design were on different aspects of the components

 ###### Dialog box
- After an invoice has been created, feedback such as 'Invoice created' should be shown to the user to communicate a successful action before the dialog box closes
- A web browser could disable pop ups and that could affect the possibility of the dialog box showing, a different component could be used instead to avoid irregularities between browsers

 ###### Main Dashboard
- The invoice list could contain a fixed header in order to describe the columns of each invoice item. The header should be fixed to enable the user visualise the information even if they have to scroll through large sets of invoice items
- A pagination feature would be a nice to have to enable the user navigate through large sets of invoice
- Once the user's cursor is on item, a hover function should be triggered in order to indicate that the user is on the specific item
- Edit and delete links could be replaced with their respective icons and a description of the icon once a user hovers over them. It would reduce the amount of text overload the user has to see while looking through the datasets
- The add invoice feature could also come with icon as a visual indicator of the action to be taken
- A nice to have would be to enable the user edit the invoice details inline the invoice display row. It would reduce the amount of pages the user would need to navigate to

6. **Application architecture**
The image below shows the structure of my folders and files for easy navigation between the various parts of the application


A downside to the current structure is that it does not include the currently logged in user details and it does not take into account sensitive information the bank payment details provides.

