Day 4: The Controller and the View
==================================

.. include:: common/original.rst.inc

Today, we are going to customize the basic ``job controller`` we created yesterday. It already has most of the code we need for Jobeet:

* A page to ``list`` all jobs
* A page to ``create`` a new job
* A page to ``update`` an existing job
* A page to ``delete`` a job

Although the code is ready to be used as is, we will refactor the templates to match closer to the `Jobeet mockups`_.

The MVC Arhitecture
---------------

For web development, the most common solution for organizing your code nowadays is the `MVC design pattern`_. In short, the MVC design pattern defines a way to organize your code according to its nature. This pattern separates the code into ``three layers``:

* The Model layer defines the business logic (the database belongs to this layer). You already know that Symfony stores all the classes and files related to the Model in the Entity/ directory of your bundles.
* The View is what the user interacts with (a template engine is part of this layer). In Symfony 2.3.2, the View layer is mainly made of Twig templates. They are stored in various Resources/views/ directories as we will see later in these lines.
* The Controller is a piece of code that calls the Model to get some data that it passes to the View for rendering to the client. When we installed Symfony at the beginning of this tutorial, we saw that all requests are managed by front controllers (app.php and app_dev.php). These front controllers delegate the real work to actions.

The Layout
----------

If you have a closer look at the `mockups`_, you will notice that much of each page looks the same. You already know that code duplication is bad, whether we are talking about HTML or PHP code, so we need to find a way to prevent these common view elements from resulting in code duplication.
One way to solve the problem is to define a header and a footer and include them in each template. A better way is to use another design pattern to solve this problem: the `decorator design pattern`_. The decorator design pattern resolves the problem the other way around: the template is decorated after the content is rendered by a global template, called a layout.
Symfony2 does not came with a default layout, so we will create one and use it to decorate our application pages.
Create a new file layout.html.twig in the src/Ibw/JobeetBundle/Resources/views/ directory and put in the following code:

src/Ibw/JobeetBundle/Resources/views/layout.html.twig

.. code-block:: html

   <!DOCTYPE html>
   <html>
       <head>
           <title>
               {% block title %}
                   Jobeet - Your best job board
               {% endblock %}
           </title>
           <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
           {% block stylesheets %}
               <link rel="stylesheet" href="{{ asset('bundles/ibwjobeet/css/main.css') }}" type="text/css" media="all" />
           {% endblock %}
           {% block javascripts %}
           {% endblock %}
           <link rel="shortcut icon" href="{{ asset('bundles/ibwjobeet/images/favicon.ico') }}" />
       </head>
       <body>
           <div id="container">
               <div id="header">
                   <div class="content">
                       <h1><a href="{{ path('ibw_job') }}">
                           <img src="{{ asset('bundles/ibwjobeet/images/logo.jpg') }}" alt="Jobeet Job Board" />
                       </a></h1>

                       <div id="sub_header">
                           <div class="post">
                               <h2>Ask for people</h2>
                               <div>
                                   <a href="{{ path('ibw_job') }}">Post a Job</a>
                               </div>
                           </div>

                           <div class="search">
                               <h2>Ask for a job</h2>
                               <form action="" method="get">
                                   <input type="text" name="keywords" id="search_keywords" />
                                   <input type="submit" value="search" />
                                   <div class="help">
                                       Enter some keywords (city, country, position, ...)
                                   </div>
                               </form>
                           </div>
                       </div>
                   </div>
               </div>

              <div id="content">
                  {% for flashMessage in app.session.flashbag.get('notice') %}
                      <div class="flash_notice">
                          {{ flashMessage }}
                      </div>
                  {% endfor %}

                  {% for flashMessage in app.session.flashbag.get('error') %}
                      <div class="flash_error">
                          {{ flashMessage }}
                      </div>
                  {% endfor %}

                  <div class="content">
                      {% block content %}
                      {% endblock %}
                  </div>
              </div>

              <div id="footer">
                  <div class="content">
                      <span class="symfony">
                          <img src="{{ asset('bundles/ibwjobeet/images/jobeet-mini.png') }}" />
                              powered by <a href="http://www.symfony.com/">
                              <img src="{{ asset('bundles/ibwjobeet/images/symfony.gif') }}" alt="symfony framework" />
                          </a>
                      </span>
                      <ul>
                          <li><a href="">About Jobeet</a></li>
                          <li class="feed"><a href="">Full feed</a></li>
                          <li><a href="">Jobeet API</a></li>
                          <li class="last"><a href="">Affiliates</a></li>
                      </ul>
                  </div>
              </div>
          </div>
      </body>
   </html>

Twig Blocks
-----------

In Twig, the default Symfony template engine, you can define blocks as we did above. A twig block can have a default content (look at the title block, for example) that can be replaced or extended in the child template as you will see in a moment.
Now, to make use of the layout we created, we will need to edit all the job templates (index, edit, new and show from src/Ibw/JobeetBundle/Resources/views/Job/) to extend the parent template (the layout) and to overwrite the content block we defined with the body block content from the original template

.. code-block:: jinja

   {% extends 'IbwJobeetBundle::layout.html.twig' %}

   {% block content %}
       <!-- original body block code goes here -->
   {% endblock %}

The Stylesheets, Images and JavaScripts
---------------------------------------

As this is not about web design, we have already prepared all the needed assets we will use for Jobeet: :download:`download the image files </resources/jobeet-images.zip>` archive and put them into the src/Ibw/JobeetBundle/Resources/public/images/ directory; :download:`download the stylesheet </resources/jobeet-css.zip>`  files archive and put them into the src/Ibw/JobeetBundle/Resources/public/css/ directory.
Now run

.. code-block:: bash

    $ php app/console assets:install web --symlink

to tell Symfony to make them available to the public.
If you look in the css folder, you will notice that we have four css files: admin.css, job.css,jobs.css and main.css. The main.css is needed in all Jobeet pages, so we included it in the layout in the stylesheet twig block. The rest are more specialized css files and we need them only in specific pages.
To add a new css file in a template, we will overwrite the stylesheet block, but call the parent before adding the new css file (so we would have the main.css and the additional css files we need).

src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig

.. code-block:: jinja

   {% extends 'IbwJobeetBundle::layout.html.twig' %}

   {% block stylesheets %}
       {{ parent() }}
       <link rel="stylesheet" href="{{ asset('bundles/ibwjobeet/css/jobs.css') }}" type="text/css" media="all" />
   {% endblock %}

   <!-- rest of the code -->

src/Ibw/JobeetBundle/Resources/views/Job/show.html.twig

.. code-block:: jinja

   {% extends 'IbwJobeetBundle::layout.html.twig' %}

   {% block stylesheets %}
       {{ parent() }}
       <link rel="stylesheet" href="{{ asset('bundles/ibwjobeet/css/job.css') }}" type="text/css" media="all" />
   {% endblock %}

   <!-- rest of the code -->

The Job Homepage Action
-----------------------

Each action is represented by a method of a class. For the job homepage, the class is JobController and the method is indexAction(). It retrieves all the jobs from the database.

src/Ibw/JobeetBundle/Controller/JobController.php

.. code-block:: php

   // ...

   public function indexAction()
   {
       $em = $this->getDoctrine()->getManager();

       $entities = $em->getRepository('IbwJobeetBundle:Job')->findAll();

       return $this->render('IbwJobeetBundle:Job:index.html.twig', array(
           'entities' => $entities
       ));
   }

   // ...

Let’s have a closer look at the code: the indexAction() method gets the Doctrine entity manager object, which is responsible for handling the process of persisting and fetching objects to and from database, and then the repository, that will create a query to retrieve all the jobs. It returns a Doctrine ArrayCollection of Job objects that are passed to the template (the View).

The Job Homepage Template
-------------------------

The index.html.twig template generates an HTML table for all the jobs. Here is the current template code:

src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig

.. code-block:: html+jinja

   {% extends 'IbwJobeetBundle::layout.html.twig' %}

   {% block stylesheets %}
       {{ parent() }}
       <link rel="stylesheet" href="{{ asset('bundles/ibwjobeet/css/jobs.css') }}" type="text/css" media="all" />
   {% endblock %}

   {% block content %}
       <h1>Job list</h1>

       <table class="records_list">
           <thead>
               <tr>
                   <th>Id</th>
                   <th>Type</th>
                   <th>Company</th>
                   <th>Logo</th>
                   <th>Url</th>
                   <th>Position</th>
                   <th>Location</th>
                   <th>Description</th>
                   <th>How_to_apply</th>
                   <th>Token</th>
                   <th>Is_public</th>
                   <th>Is_activated</th>
                   <th>Email</th>
                   <th>Expires_at</th>
                   <th>Created_at</th>
                   <th>Updated_at</th>
                   <th>Actions</th>
               </tr>
           </thead>
           <tbody>
           {% for entity in entities %}
               <tr>
                   <td><a href="{{ path('ibw_job_show', { 'id': entity.id }) }}">{{ entity.id }}</a></td>
                   <td>{{ entity.type }}</td>
                   <td>{{ entity.company }}</td>
                   <td>{{ entity.logo }}</td>
                   <td>{{ entity.url }}</td>
                   <td>{{ entity.position }}</td>
                   <td>{{ entity.location }}</td>
                   <td>{{ entity.description }}</td>
                   <td>{{ entity.howtoapply }}</td>
                   <td>{{ entity.token }}</td>
                   <td>{{ entity.ispublic }}</td>
                   <td>{{ entity.isactivated }}</td>
                   <td>{{ entity.email }}</td>
                   <td>{% if entity.expiresat %}{{ entity.expiresat|date('Y-m-d H:i:s') }}{% endif%}</td>
                   <td>{% if entity.createdat %}{{ entity.createdat|date('Y-m-d H:i:s') }}{% endif%}</td>
                   <td>{% if entity.updatedat %}{{ entity.updatedat|date('Y-m-d H:i:s') }}{% endif%}</td>
                   <td>
                       <ul>
                           <li>
                               <a href="{{ path('ibw_job_show', { 'id': entity.id }) }}">show</a>
                           </li>
                           <li>
                               <a href="{{ path('ibw_job_edit', { 'id': entity.id }) }}">edit </a>
                           </li>
                       </ul>
                   </td>
               </tr>
           {% endfor %}
           </tbody>
       </table>

       <ul>
           <li>
               <a href="{{ path('ibw_job_new') }}">
                   Create a new entry
               </a>
           </li>
       </ul>
   {% endblock %}

Let’s clean this up a bit to only display a sub-set of the available columns. Replace the twig block content with the one below:

.. code-block:: html+jinja

   {% block content %}
       <div id="jobs">
           <table class="jobs">
               {% for entity in entities %}
                   <tr class="{{ cycle(['even', 'odd'], loop.index) }}">
                       <td class="location">{{ entity.location }}</td>
                       <td class="position">
                           <a href="{{ path('ibw_job_show', { 'id': entity.id }) }}">
                               {{ entity.position }}
                           </a>
                       </td>
                       <td class="company">{{ entity.company }}</td>
                   </tr>
               {% endfor %}
           </table>
       </div>
   {% endblock %}

.. image:: /images/Day-4-2-jobs.png

The Job Page Template
---------------------

Now let’s customize the template of the job page. Open the show.html.twig file and replace its content with the following code:

src/Ibw/JobeetBundle/Resources/views/Job/show.html.twig

.. code-block:: html+jinja

   {% extends 'IbwJobeetBundle::layout.html.twig' %}

   {% block title %}
       {{ entity.company }} is looking for a {{ entity.position }}
   {% endblock %}

   {% block stylesheets %}
       {{ parent() }}
       <link rel="stylesheet" href="{{ asset('bundles/ibwjobeet/css/job.css') }}" type="text/css" media="all" />
   {% endblock %}

   {% block content %}
       <div id="job">
           <h1>{{ entity.company }}</h1>
           <h2>{{ entity.location }}</h2>
           <h3>
               {{ entity.position }}
               <small> - {{ entity.type }}</small>
           </h3>

           {% if entity.logo %}
               <div class="logo">
                   <a href="{{ entity.url }}">
                       <img src="/uploads/jobs/{{ entity.logo }}"
                           alt="{{ entity.company }} logo" />
                   </a>
               </div>
           {% endif %}

           <div class="description">
               {{ entity.description|nl2br }}
           </div>

           <h4>How to apply?</h4>

           <p class="how_to_apply">{{ entity.howtoapply }}</p>

           <div class="meta">
               <small>posted on {{ entity.createdat|date('m/d/Y') }}</small>
           </div>

           <div style="padding: 20px 0">
               <a href="{{ path('ibw_job_edit', { 'id': entity.id }) }}">
                   Edit
               </a>
           </div>
       </div>
   {% endblock %}

.. image:: /images/Day-4-individual-job.png

The Job Page Action
-------------------

The job page is generated by the show action, defined in the showAction() method of the JobController:

src/Ibw/JobeetBundle/Controller/JobController.php

.. code-block:: php

   public function showAction($id)
   {
       $em = $this->getDoctrine()->getManager();

       $entity = $em->getRepository('IbwJobeetBundle:Job')->find($id);

       if (!$entity) {
           throw $this->createNotFoundException('Unable to find Job entity.');
       }

       $deleteForm = $this->createDeleteForm($id);

       return $this->render('IbwJobeetBundle:Job:show.html.twig', array(
           'entity' => $entity,
           'delete_form' => $deleteForm->createView(),
       ));
   }

As in the index action, the IbwJobeetBundle repository class is used to retrieve a job, this time using the find() method. The parameter of this method is the unique identifier of a job, its primary key. The next section will explain why the $id parameter of the actionShow() function contains the job primary key.
If the job does not exist in the database, we want to forward the user to a 404 page, which is exactly what the throw $this->createNotFoundException() does.
As for exceptions, the page displayed to the user is different in the prod environment and in the dev ennvironment.

.. image:: /images/Day-4-error1.png
.. image:: /images/Day-4-error2.png

That’s all for today! Tomorrow we will get you familiar with the routing features.

.. include:: common/license.rst.inc

.. _`Jobeet mockups`: http://symfony.com/legacy/doc/jobeet/1_4/en/02?orm=Doctrine#chapter_02_the_project_user_stories
.. _`MVC design pattern`: http://en.wikipedia.org/wiki/Model-view-controller
.. _`mockups`: http://symfony.com/legacy/doc/jobeet/1_4/en/02?orm=Doctrine#chapter_02_the_project_user_stories
.. _`decorator design pattern`: http://en.wikipedia.org/wiki/Decorator_pattern
