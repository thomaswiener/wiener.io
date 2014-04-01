---
layout: post
title: symfony2::Form Validation for REST APIs
---

### Validation, sure! But don't reinvent the wheel

symfony2 itself is already so much fun. But understanding and working with the form component makes it even better.
Unfortunately it is probably one of the hardest parts in symfony2. But don't worry, help is on the way.

This article will describe how to use the form validation when your data is coming from a REST API Request.
When working on the Endpoints of our new API, I had to look closely under the hood of form validation.
And this is what I found out...

### Basic Requirements

* Create a REST API Endpoints that validate the POST parameter with the inbuild Form Validation of symfony2
* Validate mandatory and optional parameters

### The API Request

As a sample request we use the creation of a user.
In order to simplify this example all security measures have been removed.

#### Definition

```nginx
POST /user
```

#### Arguments

* __email__ _required_
* __password__ _required_
* __country__ _required_ _DE_ or _CH_
* __firstname__ _required_
* __lastname__ _required_
* __gender__ _optional_
  _male_ or _female_

We have a bunch of required parameters like _email_ and _name_ and also an optional one named _gender_.

The constrains of the required params are pretty straight forward.

 * all: NOT NULL
 * email: EMAIL
 * country: CHOICE (DE or CH)

_gender_ is kinda special because the form will only be valid,

* if it IS NOT in the request or
* if it IS in the request and either __male__ or __female__, but __NOT NULL__

but we will discuss this later on.

Lets focus on the controller that receives the request. Usually what you would do,
if the request came from a form, would look like the following.

```nginx
# create User Entity
$user = new User();
# create Form
$form = $this->createForm(new UserType(), $user);
# handle Request
$form->handleRequest($request);
# check if valid
if ($form->isValid()) {
    # perform some action
}

```

with the form type looking like this:

```nginx

class UserType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('email')
            ->add('password')
            ->add('country')
            ->add('firstname')
            ->add('lastname')
            ->add('gender');
    }

    public function getName()
    {
        return 'user';
    }
    ....
}

```

nothing special about that so far.


### Static Form Building

For now we take care of all required params.


```nginx
    /**
     * Create a single user
     *
     * @param Request $request
     * @param $_format
     * @throws \Symfony\Component\HttpKernel\Exception\HttpException
     * @return \Symfony\Component\HttpFoundation\Response
     */
    public function createAction(Request $request, $_format)
    {
        $user = new User();
        //reset params as if submitted via form
        $request->request->set('user', $request->request->all());
        //do regular form binding and validation
        $form = $this->createForm(new UserType(), $user);
        $form->handleRequest($request);

        if (!$form->isValid()) {
            throw new HttpException(
                AbstractController::HTTP_STATUS_UNPROCESSABLE_ENTITY,
                $this->getErrors($form)
            );
        }
        # do other magic stuff like persisting
        ....
```

### Dynamic Form Building

### Sum it up



