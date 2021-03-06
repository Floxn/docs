---
title: "Content Elements"
description: "Contao's fundamental content blocks."
aliases:
    - /documentation/content-elements/
    - /framework/content-elements/
---

In Contao, Content Elements are the fundamental content blocks. In its simplest
form it is a fragment controller which receives data in form of a content model
and returns a response.


## Definition

To create a new content element, the following things must be defined and implemented:

* __Fragment Controller__<br>
  The actual implementation of the content element is done via a class that extends
  from `AbstractContentElementController` of the Contao core.

* __Service Tag__<br>
  To identify the controller as a Contao content element, the service must be tagged
  with service tag `contao.content_element`.

  * __Type__<a id="type"></a><br>
    The *type* of a content element is a specifig string which is used to identify
    the element's template and DCA palette. The `type` can be set in the service 
    tag. If ommitted the type will be automatically generated by converting the 
    class name of the controller from pascal case to snake case and removing a possible 
    `Controller` postfix.
  
  * __Category__<br>
    All content elements are categorised within the type dropdown of the content element's
    palette. A `category` must be defined in the service tag for each content element.

* __Template__<br>
  The template name follows the naming convention mentioned beforehand. It prepends
  the *type* of the element with the prefix `ce_`.


## Example

Usually a content element is based on a specific [palette][2] in the `tl_content`
DCA configuration.

```php
// contao/dca/tl_content.php
$GLOBALS['TL_DCA']['tl_content']['palettes']['my_content_element'] = 
    '{type_legend},type;{text_legend},text'
;
```

This very simple palette enables a back end user to fill the (pre-existing) field 
`text` via the create and edit view of this content element.

The controller for this content element could look like this:

```php
// src/Controller/ContentElement/MyContentElementController.php
namespace App\Controller\ContentElement;

use Contao\ContentModel;
use Contao\CoreBundle\Controller\ContentElement\AbstractContentElementController;
use Contao\CoreBundle\ServiceAnnotation\ContentElement;
use Contao\Template;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

/**
 * @ContentElement(category="texts")
 */
class MyContentElementController extends AbstractContentElementController
{
    protected function getResponse(Template $template, ContentModel $model, Request $request): ?Response
    {
        $template->text = $model->text;
        
        return $template->getResponse();
    }
}
```

In this example the service tag was implemented via annotations.

Using the naming convention for templates mentioned above, the final template name
for this content element will be `ce_my_content_element`:

```html
<!-- templates/ce_my_content_element.html5 -->
<div class="my-content-element">    
    <?= $this->text; ?>
</div>
```

A template instance of this template will automatically be generated and passed 
to the controller's main method. The controller simply returns the parsed template
as a response.


## Options

The `contao.content_element` tag can be configured further more. The following
options are available.

| Option   | Type      | Description                                                                                         |
| -------- | --------- | ----------------------------------------------------------------------------------------------------|
| name     | `string`  | Must be `contao.content_element`.                                                                   |
| category | `string`  | Defines in which option group this content element will be placed in the content element selector.  |
| type     | `string`  | _Optional:_ The *type* mentioned in [Type]({{< ref "#type" >}}) can be customized.                  |
| renderer | `string`  | _Optional:_ The renderer can be changed to `esi`. Defaults to `inline`.                             |
| method   | `integer` | _Optional:_  Which method should be invoked on the controller.                                      |

A more complex example of a Content Element could look like this.

```yaml
# config/services.yaml
services:
    App\Controller\ContentElement\MyContentElementController:
        tags:
            -
                name: contao.content_element
                category: texts
                method: getCustomResponse
                renderer: esi
                type: my_custom_type
```


## Translations

In order to have a nice label in the back end, we also need to add a translation
for our content element - otherwise it will only be named *my_content_element*.
The translation needs to be set as follows:

```php
// contao/languages/en/default.php
$GLOBALS['TL_LANG']['CTE']['my_content_element'] = [
    'My Content Element', 
    'A Content Element for testing purposes.',
];
```


## Read more

* [DCA Configuration reference][1]
* [Manipulate and create palettes][2]
* [Create and use templates][3]
* [Customize Caching][4]


[1]: ../../reference/dca/reference
[2]: ../../reference/dca/palettes
[3]: ../templates
[4]: ../caching
