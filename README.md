# React Redux Spa Coding Convention

_Implement modular and transparent Front End Application_

## React

[General Component](react/general.md)

## Thanks

[shimohq/react-cookbook](https://github.com/shimohq/react-cookbook) provide the good naming convention part for react

## File naming

Upper camel case

## Crud Related Component naming

ElementCreate
ElementEdit
ElementList
ElementListItem
ElementDelete(the delete button)

## Pass props

Partial
{...\_.pick(this.props, [
    'currentStyle',
    'editorVisibleFontSizes',
    'currentEditFontSize',
    'setCurrentEditFontSize',
    'changeActiveElementStyle'
])}

All
{...}

Specific
currentEditFontSize={'22'}

## Naming convention

if vnode variable exist,must named in

elementVNode

elements (array)
elementsByKey,e.g elementsById(collection by id field)
