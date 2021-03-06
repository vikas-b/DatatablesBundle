# Columns

1. [Column](#1-column)
2. [Boolean Column](#2-boolean-column)
3. [Virtual Column](#3-virtual-column)
4. [Action Column](#4-action-column)
5. [Multiselect Column](#5-multiselect-column)

## 1. Column

Represents the most basic column.

### Default template

SgDatatablesBundle:column:column.html.twig

### Options

With 'null' initialized options uses the default value of the DataTables plugin.

| Option          | Type               | Default           | Required | Description |
|-----------------|--------------------|-------------------|----------|-------------|
| cell_type       | null or string     | null              |          | Cell type to be created for a column. |
| class_name      | null or string     | null              |          | Class to assign to each cell in the column. |
| content_padding | null or string     | null              |          | Add padding to the text content used when calculating the optimal with for a table. |
| data            | null or string     |                   |          | The data source for the column. |
| default_content | null or string     | null              |          | Set default, static, content for a column. |
| name            | null or string     | null              |          | Set a descriptive name for a column. |
| orderable       | bool               | true              |          | Enable or disable ordering on this column. |
| order_data      | null, array or int | null              |          | Define multiple column ordering as the default order for a column. |
| order_sequence  | null or array      | null              |          | Order direction application sequence. |
| searchable      | bool               | true              |          | Enable or disable filtering on the data in this column. |
| title           | null or string     | null              |          | Set the column title. |
| visible         | bool               | true              |          | Enable or disable the display of this column. |
| width           | null or string     | null              |          | Column width assignment. |
| add_if          | null or Closure    | null              |          | Add column only if conditions are TRUE. |
| join_type       | string             | 'leftJoin'        |          | Join type (default: 'leftJoin'), if the column represents an association. |
| type_of_field   | null or string     | null (autodetect) |          | Set the data type itself for ordering (example: integer instead string). |
| filter          | array              | TextFilter        |          | A Filter instance for individual filtering. |
| editable        | bool               | false             |          | Enable edit mode for this column. |
| editable_if     | null or Closure    | null              |          | Enable edit mode for this column only if conditions are TRUE. |

### Example

``` php
$this->columnBuilder
    ->add('title', Column::class, array(
        'title' => 'Title',
        'searchable' => true,
        'orderable' => true,
        'filter' => array(SelectFilter::class, array(
            'multiple' => true,
            'cancel_button' => true,
            'select_search_types' => array(
                '' => null,
                '2' => 'like',
                '1' => 'eq',
                'send_isNull' => 'isNull',
                'send_isNotNull' => 'isNotNull'
            ),
            'select_options' => array(
                '' => 'Any',
                '2' => 'Title with the digit 2',
                '1' => 'Title with the digit 1',
                'send_isNull' => 'is Null',
                'send_isNotNull' => 'is not Null'
            ),
        )),
        'type_of_field' => 'integer', // If the title consists only of digits.
        'add_if' => function() {
            return $this->authorizationChecker->isGranted('ROLE_USER');
        },
    ))
;
```

**For many-to-one associations:**

``` php
$users = $this->em->getRepository('AppBundle:User')->findAll();

$this->columnBuilder
    ->add('createdBy.username', Column::class, array(
        'title' => 'Created by',
        'searchable' => true,
        'orderable' => true,
        'filter' => array(SelectFilter::class, array(
            'select_options' => array('' => 'All') + $this->getOptionsArrayFromEntities($users, 'username', 'username'),
            'search_type' => 'eq'
        ))
    ))
;
```

**For one-to-many associations:**

``` php
$this->columnBuilder
    ->add('comments.title', Column::class, array(
        'title' => 'Comments',
        'data' => 'comments[,].title',
        'searchable' => true,
        'orderable' => true,
    ))
;
```

___

## 2. Boolean column

Represents a column, optimized for boolean values.

### Default template

SgDatatablesBundle:column:column.html.twig

### Options

All options of [Column](#1-column).

The option `filter` is set to `SelectFilter` by default.

**Additional:**

| Option      | Type               | Default | Required | Description     |
|-------------|--------------------|---------|----------|-----------------|
| true_icon   | null or string     | null    |          | Icon for true   |
| false_icon  | null or string     | null    |          | Icon for false  |
| true_label  | null or string     | null    |          | Label for true  |
| false_label | null or string     | null    |          | Label for false |

### Example

``` php
$this->columnBuilder
    ->add('visible', BooleanColumn::class, array(
        'title' => 'Visible',
        'true_icon' => 'glyphicon glyphicon-ok',
        'false_icon' => 'glyphicon glyphicon-remove',
        'true_label' => 'yes',
        'false_label' => 'no',
    ))
;
```
___

## 3. Virtual column

Represents a virtual column.

### Default template

SgDatatablesBundle:Column:column.html.twig

### Options

All options of [Column](#1-column), except `data` and `join_type`.

The options `searchable` and `orderable` are set to `false` by default.

**Additional:**

| Option        | Type               | Default | Required | Description     |
|---------------|--------------------|---------|----------|-----------------|
| order_column  | null or string     | null    |          | The name of an existing column that is used for ordering. |
| search_column | null or string     | null    |          | The name of an existing column that is used for searching. |

### Example

``` php
public function getLineFormatter()
{
    $formatter = function($row) {
        $row['test'] = 'Post from ' . $row['createdBy']['username'];

        return $row;
    };

    return $formatter;
}

public function buildDatatable(array $options = array())
{
    // ...

    $users = $this->em->getRepository('AppBundle:User')->findAll();

    $this->columnBuilder
        ->add('test', VirtualColumn::class, array(
            'title' => 'Test virtual',
            'searchable' => true,
            'orderable' => true,
            'order_column' => 'createdBy.username', // use the 'createdBy.username' column for ordering
            'search_column' => 'createdBy.username', // use the 'createdBy.username' column for searching
        ))
        ->add('createdBy.username', Column::class, array(
            'title' => 'Created by',
            'searchable' => true,
            'orderable' => true,
            'filter' => array(SelectFilter::class, array(
                'select_options' => array('' => 'All') + $this->getOptionsArrayFromEntities($users, 'username', 'username'),
                'search_type' => 'eq'
            ))
        ))

        // ...
    ;
}
```
___

## 4. Action column

A Column to display CRUD action labels or buttons.

### Default template

SgDatatablesBundle:Column:column.html.twig

### Action column options

| Option          | Type               | Default           | Required | Description |
|-----------------|--------------------|-------------------|----------|-------------|
| cell_type       | null or string     | null              |          | Cell type to be created for a column. |
| class_name      | null or string     | null              |          | Class to assign to each cell in the column. |
| content_padding | null or string     | null              |          | Add padding to the text content used when calculating the optimal with for a table. |
| name            | null or string     | null              |          | Set a descriptive name for a column. |
| title           | null or string     | null              |          | Set the column title. |
| visible         | bool               | true              |          | Enable or disable the display of this column. |
| width           | null or string     | null              |          | Column width assignment. |
| add_if          | null or Closure    | null              |          | Add column only if conditions are TRUE. |
| actions         | array              |                   | X        | Contains all the actions. |
| start_html      | null or string     | null              |          | HTML code before all actions. |
| end_html        | null or string     | null              |          | HTML code after all actions. |

### Action options

| Option           | Type            | Default | Required | Description |
|------------------|-----------------|---------|----------|-------------|
| route            | string          |         | X        | The name of the Action route. |
| route_parameters | null or array   | null    |          | The route parameters. |
| icon             | null or string  | null    |          | An icon for the Action. |
| label            | null or string  | null    |          | A label for the Action. |
| confirm          | bool            | false   |          | Show confirm message if true. |
| confirm_message  | null or string  | null    |          | The confirm message. |
| attributes       | null or array   | null    |          | HTML <a> Tag attributes (except 'href'). |
| render_if        | null or Closure | null    |          | Render an Action only if conditions are TRUE. |
| start_html       | null or string  | null    |          | HTML code before the <a> Tag. |
| end_html         | null or string  | null    |          | HTML code after the <a> Tag. |

### Example

**Don't forget to add the following to your route annotation:**

``` php
options = {"expose" = true}
```

#### The Controller

``` php
/**
 * Finds and displays a Post entity.
 *
 * @param Post $post
 *
 * @Route("/{_locale}/{id}.{_format}", name = "post_show", options = {"expose" = true})
 * @Method("GET")
 * @Security("has_role('ROLE_USER')")
 *
 * @return Response
 */
public function showAction(Post $post)
{
    $deleteForm = $this->createDeleteForm($post);

    return $this->render('post/show.html.twig', array(
        'post' => $post,
        'delete_form' => $deleteForm->createView(),
    ));
}
```

#### The Datatable Class

``` php
$this->columnBuilder
    ->add(null, ActionColumn::class, array(
        'title' => 'Actions',
        'start_html' => '<div class="start_actions">',
        'end_html' => '</div>',
        'actions' => array(
            array(
                'route' => 'post_show',
                'label' => 'Show Posting',
                'route_parameters' => array(
                    'id' => 'id',
                    '_format' => 'html',
                    '_locale' => 'en'
                ),
                'render_if' => function($row) {
                    return $row['createdBy']['username'] === 'user' && $this->authorizationChecker->isGranted('ROLE_USER');
                },
                'attributes' => array(
                    'rel' => 'tooltip',
                    'title' => 'Show',
                    'class' => 'btn btn-primary btn-xs',
                    'role' => 'button'
                ),
                'start_html' => '<div class="start_show_action">',
                'end_html' => '</div>',
            )
        )
    ))
;
```

___

## 5. Multiselect column

Support for Bulk Actions.

### Default template

SgDatatablesBundle:Column:multiselect.html.twig

### Options

All options of [Action Column](#4-action-column), except `title`.

**Additional Column options:**

| Option               | Type               | Default | Required | Description     |
|----------------------|--------------------|---------|----------|-----------------|
| attributes           | null or array      | null    |          | HTML <input> Tag attributes (except 'type' and 'value'). |
| value                | string             | 'id'    |          | A checkbox value, generated by column name. |
| render_actions_to_id | null or string     | null    |          | Id selector where all multiselect actions are rendered. |
| render_if            | null or Closure    | null    |          | Render a Checkbox only if conditions are TRUE. |

### Example

#### The Controller

``` php
/**
 * Bulk delete action.
 *
 * @param Request $request
 *
 * @Route("/bulk/delete", name="post_bulk_delete")
 * @Method("POST")
 *
 * @return Response
 */
public function bulkDeleteAction(Request $request)
{
    $isAjax = $request->isXmlHttpRequest();

    if ($isAjax) {
        $choices = $request->request->get('data');
        $token = $request->request->get('token');

        if (!$this->isCsrfTokenValid('multiselect', $token)) {
            throw new AccessDeniedException('The CSRF token is invalid.');
        }

        $em = $this->getDoctrine()->getManager();
        $repository = $em->getRepository('AppBundle:Post');

        foreach ($choices as $choice) {
            $entity = $repository->find($choice['id']);
            $em->remove($entity);
        }

        $em->flush();

        return new Response('Success', 200);
    }

    return new Response('Bad Request', 400);
}
```

#### The Datatable Class

**If the MultiselectColumn is the first Column (position: 0), it is not orderable. Change the initial order statement:**

``` php
$this->options->set(array(
    'order' => array(array(1, 'asc')),
));

$this->columnBuilder
    ->add(null, MultiselectColumn::class, array(
        'start_html' => '<div class="start_checkboxes">',
        'end_html' => '</div>',
        'add_if' => function() {
            // add Column if condition is true
            return $this->authorizationChecker->isGranted('ROLE_USER');
        },
        //'attributes' => array('data-toggle' => 'toggle'),
        'value' => 'id',
        'render_actions_to_id' => 'sidebar-multiselect-actions', // custom Dom id for the actions
        'render_if' => function($row) {
            // render checkbox only if the Post created by 'user'
            return $row['createdBy']['username'] === 'user';
        },
        'actions' => array(
            array(
                'route' => 'post_bulk_delete',
                'icon' => 'glyphicon glyphicon-ok',
                'label' => 'Delete Postings',
                'attributes' => array(
                    'rel' => 'tooltip',
                    'title' => 'Delete',
                    'class' => 'btn btn-primary btn-xs',
                    'role' => 'button'
                ),
                'confirm' => true,
                'confirm_message' => 'Really?',
                'start_html' => '<div class="start_delete_action">',
                'end_html' => '</div>',
                'render_if' => function(/* the $row argument is not available in this context */) {
                    return $this->authorizationChecker->isGranted('ROLE_USER');
                }
            ),
        )
    ))
```
___

