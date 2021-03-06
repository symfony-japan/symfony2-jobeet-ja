19日目: 国際化と地域化
======================

.. include:: common/original.rst.inc

| 昨日は、検索エンジンの機能に、 AJAX の便利さを追加して、より楽しくしました。
| 今日は、 Jobeet の国際化（または i18n ）と地域化（または l10n ）についてお話します。

..
   Yesterday, we finished the search engine feature by making it more fun with the addition of some AJAX goodness.
   Now, we will talk about Jobeet internationalization (or i18n) and localization (or l10n).

.. note::

   Wikipedia から:
      * 国際化( アメリカ英語: internationalization イギリス英語: internationalisation、i18n )は、ソフトウェアに技術的な変更を加えることなく多様な言語や地域に適合できるようにする、ソフトウェア設計の工程である。
      * 地域化（アメリカ英語: localization イギリス英語: localisation、L10N）は、地域固有の構成部品や翻訳テキストを追加することによって、ソフトウェアを特定の地域や言語に適合させる工程である。

      ..
         * Internationalization is the process of designing a software application so that it can be adapted to various languages and regions without engineering changes.
         * Localization is the process of adapting software for a specific region or language by adding locale-specific components and translating text.

ユーザー
--------

| ユーザー無しには国際化は不可能です。
| ウェブサイトが複数の言語で、または、世界のさまざまな地域のために使用可能になると、ユーザーは自分に最も適したものを選択する責任があります。
| Symfony の国際化と地域化の機能はユーザーの culture に基づいています。 culture はユーザーの言語と国の組み合わせです。
| たとえば、フランス語を話すユーザーカルチャーは FR で、フランスからのユーザーのためのカルチャーはは fr_FR です。
| 翻訳は、Translator サービスによって処理され、ユーザーのロケールを使用して検索し、翻訳されたメッセージを返し ます。
| それを使用する前に、設定ファイルの ``translator`` を有効にします。

..
   No internationalization is possible without a user.
   When your website is available in several languages or for different regions of the world, the user is responsible for choosing the one that fits him best.
   The i18n and l10n features of symfony are based on the user culture.
   The culture is the combination of the language and the country of the user.
   For instance, the culture for a user that speaks French is fr and the culture for a user from France is fr_FR.
   Translations are handled by a Translator service that uses the user’s locale to lookup and return translated messages.
   Before using it, enable the Translator in your configuration:

app/config/config.yml

.. code-block:: yaml

   # ...

   framework:
       #esi:             ~
       translator:      { fallback: en }
       # ...
       default_locale:  "en"

   # ...

URL内のカルチャー
-----------------

| Jobeet の Web サイトは英語とフランス語の両方で利用できます。
| URL は単独のリソースのみを表しますので、カルチャーは URL に埋め込まれている必要があります。
| そのためには、 routing.yml ファイルを開き、API以外のすべてのルートに特別なロケール変数を追加します。
| シンプルなルートにするため、URLの先頭に /{_locale} を追加します。

..
   The Jobeet website will be available in English and French.
   As an URL can only represent a single resource, the culture must be embedded in the URL.
   In order to do that, open the routing.yml file, and add the special locale variable for all routes but the api.
   For simple routes, add /{_locale} to the front of the url:

src/Ibw/JobeetBundle/Resources/config/routing.yml

.. code-block:: yaml

   login:
       pattern: /login
       defaults: { _controller: IbwJobeetBundle:Default:login }

   login_check:
       pattern: /login_check

   logout:
     pattern: /logout

   IbwJobeetBundle_category:
       pattern:  /{_locale}/category/{slug}/{page}
       defaults: { _controller: IbwJobeetBundle:Category:show, page: 1 }
       requirements:
           _locale: en|fr

   IbwJobeetBundle_job:
       resource: "@IbwJobeetBundle/Resources/config/routing/job.yml"
       prefix:   /{_locale}/job
       requirements:
           _locale: en|fr

   ibw_jobeet_homepage:
       pattern:  /
       defaults: { _controller: IbwJobeetBundle:Job:index }

   IbwJobeetBundle_api:
       pattern: /api/{token}/jobs.{_format}
       defaults: {_controller: "IbwJobeetBundle:Api:list"}
       requirements:
           _format: xml|json|yaml

   IbwJobeetBundle_ibw_affiliate:
       resource: "@IbwJobeetBundle/Resources/config/routing/affiliate.yml"
       prefix:   /{_locale}/affiliate
       requirements:
           _locale: en|fr

| 言語の数だけホームページでサポートする必要があるので（/en/, /fr/, …）、デフォルトのホームページ（ / ）は、ユーザーの culture に従って、適切なローカライズされたものにリダイレクトする必要があります。
| 初めての訪問で、まだユーザーがカルチャーを持っていない場合、優先カルチャーが選択されます。（以前にそれを宣言したように、en になります）。
| では、実際に新しいルートの追加、および、ホームページのルートの変更をしてみましょう。

..
   As we need as many homepages as languages we support (/en/, /fr/, …), the default homepage (/) must redirect to the appropriate localized one, according to the user culture.
   But if the user has no culture yet, because he comes to Jobeet for the first time, the preferred culture will be chosen for him (as we declare it previously, it will be en).
   So go ahead and add a new route to do that for you and modify your homepage route also :

src/Ibw/JobeetBundle/Resources/config/routing.yml

.. code-block:: yaml

   # ...
   ibw_jobeet_homepage:
       pattern:  /{_locale}/
       defaults: { _controller: IbwJobeetBundle:Job:index }
       requirements:
           _locale: en|fr
   # ...

   IbwJobeetBundle_nonlocalized:
       pattern:  /
       defaults: { _controller: "IbwJobeetBundle:Job:index" }

コントローラ内のこれら二つのルートの動作を追加および変更してみましょう。

.. Let’s add and modify the behaviour of these two routes in the controller:

src/Ibw/JobeetBundle/Controller/JobController.php

.. code-block:: php

   // ...

       public function indexAction()
       {
           $request = $this->getRequest();

           if($request->get('_route') == 'IbwJobeetBundle_nonlocalized') {
               return $this->redirect($this->generateUrl('ibw_jobeet_homepage'));
           }

           $em = $this->getDoctrine()->getManager();

           // ...
       }

   // ...

ユーザがカルチャーを指定せずにJobeetのプラットフォームにアクセスする場合は（http：//jobeet.local/app_dev.php）、デフォルトのカルチャーを持つホームページにリダイレクトされます(http://jobeet.local/app_dev.php/en/)。

..
   If the user will access the Jobeet platform without specifying his preffered culture (http://jobeet.local/app_dev.php),
   he will be redirected to the homepage having the culture selected by default for him (http://jobeet.local/app_dev.php/en/).

カルチャーのテスト
------------------

| 実装のテストをする時間です。
| しかし、多くのテストを追加する前に、既存のものを修正する必要があります。
| すべての URL が変更されているように、すべての機能テストのファイルを編集して、 URL の前に /en 追加します。

..
   It is time to test our implementation.
   But before adding more tests, we need to fix the existing ones.
   As all URLs have changed, edit all functional test files and add /en in front of all URLs.

src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Tests\Controller;

   use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
   use Symfony\Bundle\FrameworkBundle\Console\Application;
   use Symfony\Component\Console\Output\NullOutput;
   use Symfony\Component\Console\Input\ArrayInput;
   use Doctrine\Bundle\DoctrineBundle\Command\DropDatabaseDoctrineCommand;
   use Doctrine\Bundle\DoctrineBundle\Command\CreateDatabaseDoctrineCommand;
   use Doctrine\Bundle\DoctrineBundle\Command\Proxy\CreateSchemaDoctrineCommand;
   use Symfony\Component\DomCrawler\Crawler;

   class JobControllerTest extends WebTestCase
   {
       // ...

       public function testIndex()
       {
           // get the custom parameters from app config.yml
           $kernel = static::createKernel();
           $kernel->boot();
           $max_jobs_on_homepage = $kernel->getContainer()->getParameter('max_jobs_on_homepage');

           $client = static::createClient();

           $crawler = $client->request('GET', '/fr/');
           $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::indexAction', $client->getRequest()->attributes->get('_controller'));

           // If the selected culture is italian, the page requested will not be found
           $crawler = $client->request('GET', '/it/');
           $this->assertTrue(404 === $client->getResponse()->getStatusCode());

           $crawler = $client->request('GET', '/en/');
           $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::indexAction', $client->getRequest()->attributes->get('_controller'));

           // expired jobs are not listed
           $this->assertTrue($crawler->filter('.jobs td.position:contains("Expired")')->count() == 0);

           // only $max_jobs_on_homepage jobs are listed for a category
           $this->assertTrue($crawler->filter('.category_programming tr')->count()<= $max_jobs_on_homepage);
           $this->assertTrue($crawler->filter('.category_design .more_jobs')->count() == 0);
           $this->assertTrue($crawler->filter('.category_programming .more_jobs')->count() == 1);

           // jobs are sorted by date
           $this->assertTrue($crawler->filter('.category_programming tr')->first()->filter(sprintf('a[href*="/%d/"]', $this->getMostRecentProgrammingJob()->getId()))->count() == 1);

           // each job on the homepage is clickable and give detailed information
           $job = $this->getMostRecentProgrammingJob();
           $link = $crawler->selectLink('Web Developer')->first()->link();
           $crawler = $client->click($link);
           $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::showAction', $client->getRequest()->attributes->get('_controller'));
           $this->assertEquals($job->getCompanySlug(), $client->getRequest()->attributes->get('company'));
           $this->assertEquals($job->getLocationSlug(), $client->getRequest()->attributes->get('location'));
           $this->assertEquals($job->getPositionSlug(), $client->getRequest()->attributes->get('position'));
           $this->assertEquals($job->getId(), $client->getRequest()->attributes->get('id'));

           // a non-existent job forwards the user to a 404
           $crawler = $client->request('GET', '/en/job/foo-inc/milano-italy/0/painter');
           $this->assertTrue(404 === $client->getResponse()->getStatusCode());

           // an expired job page forwards the user to a 404
           $crawler = $client->request('GET', sprintf('/en/job/sensio-labs/paris-france/%d/web-developer', $this->getExpiredJob()->getId()));
           $this->assertTrue(404 === $client->getResponse()->getStatusCode());
       }

       public function testJobForm()
       {
           $client = static::createClient();
           $crawler = $client->request('GET', '/en/job/new');

           $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::newAction', $client->getRequest()->attributes->get('_controller'));

           $form = $crawler->selectButton('Preview your job')->form(array(
               'job[company]'      => 'Sensio Labs',
               'job[url]'          => 'http://www.sensio.com',
               'job[file]'         => __DIR__.'/../../../../../web/bundles/ibwjobeet/images/sensio-labs.gif',
               'job[how_to_apply]' => 'Send me an email',
               'job[description]'  => 'You will work with symfony to develop websites for our customers',
               'job[location]'     => 'Atlanta, USA',
               'job[email]'        => 'for.a.job@example.com',
               'job[position]'     => 'Developer',
               'job[is_public]'    => false,
           ));

           $client->submit($form);
           $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::createAction', $client->getRequest()->attributes->get('_controller'));

           $client->followRedirect();
           $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::previewAction', $client->getRequest()->attributes->get('_controller'));

           $kernel = static::createKernel();
           $kernel->boot();
           $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');

           $query = $em->createQuery('SELECT count(j.id) from IbwJobeetBundle:Job j WHERE j.location = :location AND j.is_activated IS NULL AND j.is_public = 0');
           $query->setParameter('location', 'Atlanta, USA');
           $this->assertTrue(0 < $query->getSingleScalarResult());

           $crawler = $client->request('GET', '/en/job/new');
           $form = $crawler->selectButton('Preview your job')->form(array(
               'job[company]'      => 'Sensio Labs',
               'job[position]'     => 'Developer',
               'job[location]'     => 'Atlanta, USA',
               'job[email]'        => 'not.an.email',
           ));
           $crawler = $client->submit($form);

           // check if we have 3 errors
           $this->assertTrue($crawler->filter('.error_list')->count() == 3);
           // check if we have error on job_description field
           $this->assertTrue($crawler->filter('#job_description')->siblings()->first()->filter('.error_list')->count() == 1);
           // check if we have error on job_how_to_apply field
           $this->assertTrue($crawler->filter('#job_how_to_apply')->siblings()->first()->filter('.error_list')->count() == 1);
           // check if we have error on job_email field
           $this->assertTrue($crawler->filter('#job_email')->siblings()->first()->filter('.error_list')->count() == 1);
       }

       public function createJob($values = array(), $publish = false)
       {
           $client = static::createClient();
           $crawler = $client->request('GET', '/en/job/new');
           $form = $crawler->selectButton('Preview your job')->form(array_merge(array(
               'job[company]'      => 'Sensio Labs',
               'job[url]'          => 'http://www.sensio.com/',
               'job[position]'     => 'Developer',
               'job[location]'     => 'Atlanta, USA',
               'job[description]'  => 'You will work with symfony to develop websites for our customers.',
               'job[how_to_apply]' => 'Send me an email',
               'job[email]'        => 'for.a.job@example.com',
               'job[is_public]'    => false,
           ), $values));

           $client->submit($form);
           $client->followRedirect();

           if($publish) {
               $crawler = $client->getCrawler();
               $form = $crawler->selectButton('Publish')->form();
               $client->submit($form);
               $client->followRedirect();
           }

           return $client;
       }

       // ...

       public function testEditJob()
       {
           $client = $this->createJob(array('job[position]' => 'FOO3'), true);
           $crawler = $client->getCrawler();
           $crawler = $client->request('GET', sprintf('/en/job/%s/edit', $this->getJobByPosition('FOO3')->getToken()));
           $this->assertTrue( 404 === $client->getResponse()->getStatusCode());
       }

       public function testExtendJob()
       {
           // A job validity cannot be extended before the job expires soon
           $client = $this->createJob(array('job[position]' => 'FOO4'), true);
           $crawler = $client->getCrawler();
           $this->assertTrue($crawler->filter('input[type=submit]:contains("Extend")')->count() == 0);

           // A job validity can be extended hen the job expires soon
           // Create a new FOO5 job
           $client = $this->createJob(array('job[position]' => 'FOO5'), true);
           // Get the job and change the expire date to today
           $kernel = static::createKernel();
           $kernel->boot();
           $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');
           $job = $em->getRepository('IbwJobeetBundle:Job')->findOneByPosition('FOO5');
           $job->setExpiresAt(new \DateTime());
           $em->flush();

           // Go to preview page and extend the job
           $crawler = $client->request('GET', sprintf('/en/job/%s/%s/%s/%s', $job->getCompanySlug(), $job->getLocationSlug(), $job->getToken(), $job->getPositionSlug()));
           $crawler = $client->getCrawler();

           $form = $crawler->selectButton('Extend')->form();
           $client->submit($form);
           $client->followRedirect();
           $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::previewAction', $client->getRequest()->attributes->get('_controller'));

           // Reload the job from database
           $job = $this->getJobByPosition('FOO5');

           // Check the expiration date
           $this->assertTrue($job->getExpiresAt()->format('y/m/d') == date('y/m/d', time() + 86400 * 30));
       }

       public function testSearch()
       {
           $client = static::createClient();

           $crawler = $client->request('GET', '/en/job/search');
           $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::searchAction', $client->getRequest()->attributes->get('_controller'));

           $crawler = $client->request('GET', '/en/job/search?query=sens*', array(), array(), array(
               'X-Requested-With' => 'XMLHttpRequest',
           ));
           $this->assertTrue($crawler->filter('tr')->count()== 2);
       }
   }

src/Ibw/JobeetBundle/Tests/Controller/AffiliateControllerTest.php

.. code-block:: php

   // ...

       public function testAffiliateForm()
       {
           $client = static::createClient();
           $crawler = $client->request('GET', '/en/affiliate/new');

           $this->assertEquals('Ibw\JobeetBundle\Controller\AffiliateController::newAction', $client->getRequest()->attributes->get('_controller'));

           $form = $crawler->selectButton('Submit')->form(array(
               'affiliate[url]'   => 'http://sensio-labs.com/',
               'affiliate[email]' => 'fabien.potencier@example.com'
           ));

           $client->submit($form);
           $this->assertEquals('Ibw\JobeetBundle\Controller\AffiliateController::createAction', $client->getRequest()->attributes->get('_controller'));

           $kernel = static::createKernel();
           $kernel->boot();
           $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');

           $crawler = $client->request('GET', '/en/affiliate/new');
           $form = $crawler->selectButton('Submit')->form(array(
               'affiliate[email]'        => 'not.an.email',
           ));
           $crawler = $client->submit($form);

           // check if we have 1 errors
           $this->assertTrue($crawler->filter('.error_list')->count() == 1);
           // check if we have error on affiliate_email field
           $this->assertTrue($crawler->filter('#affiliate_email')->siblings()->first()->filter('.error_list')->count() == 1);
       }

       public function testCreate()
       {
           $client = static::createClient();
           $crawler = $client->request('GET', '/en/affiliate/new');
           $form = $crawler->selectButton('Submit')->form(array(
               'affiliate[url]'   => 'http://sensio-labs.com/',
               'affiliate[email]' => 'address@example.com'
           ));

           $client->submit($form);
           $client->followRedirect();

           $this->assertEquals('Ibw\JobeetBundle\Controller\AffiliateController::waitAction', $client->getRequest()->attributes->get('_controller'));

           return $client;
       }

       public function testWait()
       {
           $client = static::createClient();
           $crawler = $client->request('GET', '/en/affiliate/wait');

           $this->assertEquals('Ibw\JobeetBundle\Controller\AffiliateController::waitAction', $client->getRequest()->attributes->get('_controller'));
       }

       // ...

src/Ibw/JobeetBundle/Tests/Controller/CategoryControllerTest.php

.. code-block:: php

   // ...

       public function testShow()
       {
           $kernel = static::createKernel();
           $kernel->boot();

           // get the custom parameters from app/config.yml
           $max_jobs_on_category = $kernel->getContainer()->getParameter('max_jobs_on_category');
           $max_jobs_on_homepage = $kernel->getContainer()->getParameter('max_jobs_on_homepage');

           $client = static::createClient();

           $categories = $this->em->getRepository('IbwJobeetBundle:Category')->getWithJobs();

           // categories on homepage are clickable
           foreach($categories as $category) {
               $crawler = $client->request('GET', '/en/');

               $link = $crawler->selectLink($category->getName())->link();
               $crawler = $client->click($link);

               $this->assertEquals('Ibw\JobeetBundle\Controller\CategoryController::showAction', $client->getRequest()->attributes->get('_controller'));
               $this->assertEquals($category->getSlug(), $client->getRequest()->attributes->get('slug'));

               $jobs_no = $this->em->getRepository('IbwJobeetBundle:Job')->countActiveJobs($category->getId());

               // categories with more than $max_jobs_on_homepage jobs also have a "more" link
               if($jobs_no > $max_jobs_on_homepage) {
                   $crawler = $client->request('GET', '/en/');
                   $link = $crawler->filter(".category_" . $category->getSlug() . " .more_jobs a")->link();
                   $crawler = $client->click($link);

                   $this->assertEquals('Ibw\JobeetBundle\Controller\CategoryController::showAction', $client->getRequest()->attributes->get('_controller'));
                   $this->assertEquals($category->getSlug(), $client->getRequest()->attributes->get('slug'));
               }

               $pages = ceil($jobs_no/$max_jobs_on_category);

               // only $max_jobs_on_category jobs are listed
               $this->assertTrue($crawler->filter('.jobs tr')->count() <= $max_jobs_on_category);
               if ($jobs_no == 1)
               {
                   $this->assertRegExp("/One job in this category/", $crawler->filter('.pagination_desc')->text());
               }else{
                   $this->assertRegExp("/" . $jobs_no . " jobs/", $crawler->filter('.pagination_desc')->text());
               }

               $this->assertRegExp("/" . $jobs_no . " jobs/", $crawler->filter('.pagination_desc')->text());

               if($pages > 1) {
                   $this->assertRegExp("/page 1\/" . $pages . "/", $crawler->filter('.pagination_desc')->text());

                   for ($i = 2; $i <= $pages; $i++) {
                       $link = $crawler->selectLink($i)->link();
                       $crawler = $client->click($link);

                       $this->assertEquals('Ibw\JobeetBundle\Controller\CategoryController::showAction', $client->getRequest()->attributes->get('_controller'));
                       $this->assertEquals($i, $client->getRequest()->attributes->get('page'));
                       $this->assertTrue($crawler->filter('.jobs tr')->count() <= $max_jobs_on_category);
                       if($jobs_no > 1) {
                           $this->assertRegExp("/" . $jobs_no . " jobs/", $crawler->filter('.pagination_desc')->text());
                       }
                       $this->assertRegExp("/page " . $i . "\/" . $pages . "/", $crawler->filter('.pagination_desc')->text());
                   }
               }
           }
       }

       // ...

言語の切り替え
--------------

| ユーザーのカルチャーを変えるために、言語形式がレイアウトに追加される必要があります。
| それを作成してみましょう。

..
   For the user to change the culture, a language form must be added in the layout.
   Let’s create it:

src/Ibw/JobeetBundle/Resources/views/layout.html.twig

.. code-block:: html+jinja

   <!-- ... -->
   <div id="footer">
       <div class="content">
           <!-- ... -->
           <form action="{{ path('IbwJobeetBundle_changeLanguage') }}" method="get">
               <label>Language</label>
                   <select name="language">
                       <option value="en" {% if app.request.get('_locale') == 'en' %}selected="selected"{% endif %}>English</option>
                       <option value="fr" {% if app.request.get('_locale') == 'fr' %}selected="selected"{% endif %}>French</option>
                   </select>
               <input type="submit" value="Ok">
           </form>
       </div>
   </div>
   <!-- ... -->

言語が変更されるアクションと一致する新しいルートを追加します。

.. Add a new route to match the action in which we will change the language:

src/Ibw/JobeetBundle/Resources/config/routing.yml

.. code-block:: yaml

   # ...

   IbwJobeetBundle_changeLanguage:
       pattern: /change_language
       defaults: { _controller: "IbwJobeetBundle:Default:changeLanguage" }

ここで、アクションを変更します。

.. Now, the action:

src/Ibw/JobeetBundle/Controller/DefaultController.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Controller;

   use Symfony\Bundle\FrameworkBundle\Controller\Controller;
   use Symfony\Component\Security\Core\SecurityContext;

   class DefaultController extends Controller
   {
       // ...

       public function changeLanguageAction()
       {
           $language = $this->getRequest()->get('language');
           return $this->redirect($this->generateUrl('ibw_jobeet_homepage', array('_locale' => $language)));
       }
   }

キャッシュをクリアすることを忘れないでください！

.. Don’ forget to clear the cache!

テンプレート
------------

| 国際化されたウェブサイトは、ユーザーインターフェイスが複数の言語に翻訳されることを意味します。
| それは、ここでは、デフォルトの英語とフランス語になります。
| テンプレートを翻訳するためには、Twig タグ ``{% trans %}`` を使用します。
| Symfony がテンプレートをレンダリングする際に ``{% trans %}`` タグを発見するたびに、現在のユーザーの culture 用の翻訳を探します。
| 翻訳が見つかった場合はそれが使用され、見つからなかった場合は、翻訳されるはずの文字列をフォールバックの値として使用します。
| すべての翻訳は、 src/Ibw/JobeetBundle/Resources/translations/ ディレクトリに配置され、カタログに格納されています。
| このために、標準で最も柔軟なものである XLIFF 形式を使用します。
| では、テンプレートに ``{% trans %}`` タグを追加することから翻訳をはじめましょう。

..
   An internationalized website means that the user interface is translated into several language.
   For us, it will be english, by default, and french.
   In order to translate the templates, we will use the Twig tag {% trans %}.
   When symfony renders a template, each time the {% trans %} tag is found, symfony looks for a translation for the current user’s culture.
   If the translation is found, it is used, if not, the string supposed to be translated is returned as a fallback value.
   All the translations are stored in a catalogue, that is located in the src/Ibw/JobeetBundle/Resources/translations/ directory.
   For this, we will use the XLIFF format, which is a standard and the most flexible one.
   Let’s start translating by adding the {% trans %} tag inside the templates:

src/Ibw/JobeetBundle/Resources/views/layout.html.twig

.. code-block:: html+jinja

   <!DOCTYPE html>
   <html>
       <head>
           <title>
               {% block title %}
                   {% trans %}Jobeet - Your best job board{% endtrans %}
               {% endblock %}
           </title>
           <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
           {% block stylesheets %}
               <link rel="stylesheet" href="{{ asset('bundles/ibwjobeet/css/main.css') }}" type="text/css" media="all" />
               <link rel="alternate" type="application/atom+xml" title="Latest Jobs" href="{{ url('ibw_job', {'_format': 'atom'}) }}" />
           {% endblock %}
           {% block javascripts %}
               <script type="text/javascript" src="{{ asset('bundles/ibwjobeet/js/jquery-2.0.3.min.js') }}"></script>
               <script type="text/javascript" src="{{ asset('bundles/ibwjobeet/js/search.js') }}"></script>
           {% endblock %}
           <link rel="shortcut icon" href="{{ asset('bundles/ibwjobeet/images/favicon.ico') }}" />
       </head>
       <body>
           <div id="container">
               <div id="header">
                   <div class="content">
                       <h1><a href="{{ path('ibw_jobeet_homepage') }}">
                           <img alt="Jobeet Job Board" src="{{ asset('bundles/ibwjobeet/images/logo.jpg') }}" />
                       </a></h1>

                       <div id="sub_header">
                           <div class="post">
                               <h2>{% trans %}Ask for people{% endtrans %}</h2>
                               <div>
                                   <a href="{{ path('ibw_job_new') }}">{% trans %}Post a Job{% endtrans %}</a>
                               </div>
                           </div>

                           <div class="search">
                               <h2>{% trans %}Ask for a job{% endtrans %}</h2>
                               <form action="{{ path('ibw_job_search') }}" method="get">
                                   <input type="text" name="query" value="{{ app.request.get('query') }}" id="search_keywords" />
                                   <input type="submit" value="search" />
                                   <img id="loader" src="{{ asset('bundles/ibwjobeet/images/loader.gif') }}" style="vertical-align: middle; display: none" />
                                   <div class="help">
                                       {% trans %}Enter some keywords (city, country, position, ...){% endtrans %}
                                   </div>
                               </form>
                           </div>
                       </div>
                   </div>
               </div>
              <div id="job_history">
                   {% trans %}Recent viewed jobs:{% endtrans %}
                   <ul>
                       {% for job in app.session.get('job_history') %}
                           <li>
                               <a href="{{ path('ibw_job_show', { 'id': job.id, 'company': job.companyslug, 'location': job.locationslug, 'position': job.positionslug }) }}">{{ job.position }} - {{ job.company }}</a>
                           </li>
                       {% endfor %}
                   </ul>
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
                          <li><a href="">{% trans %}About Jobeet{% endtrans %}</a></li>
                          <li class="feed"><a href="{{ path('ibw_job', {'_format': 'atom'}) }}">{% trans %}Full feed{% endtrans %}</a></li>
                          <li><a href="">{% trans %}Jobeet API{% endtrans %}</a></li>
                          <li class="last"><a href="{{ path('ibw_affiliate_new') }}">{% trans %}Become an affiliate{% endtrans %}</a></li>
                      </ul>
                      <form action="{{ path('IbwJobeetBundle_changeLanguage') }}" method="get">
                          <label>{% trans %}Language{% endtrans %}</label>
                          <select name="language">
                              <option value="en" {% if app.request.get('_locale') == 'en' %}selected="selected"{% endif %}>English</option>
                                   <option value="fr" {% if app.request.get('_locale') == 'fr' %}selected="selected"{% endif %}>French</option>
                          </select>
                          <input type="submit" value="Ok">
                      </form>
                  </div>
              </div>
          </div>
      </body>
   </html>

src/Ibw/JobeetBundle/Resources/views/Job/show.html.twig

.. code-block:: html+jinja

   {% extends 'IbwJobeetBundle::layout.html.twig' %}

   {% block title %}
       {% trans with {'%company%': entity.company, '%position%': entity.position} %}%company% is looking for a %position%{% endtrans %}
   {% endblock %}

   {% block stylesheets %}
       {{ parent() }}
       <link rel="stylesheet" href="{{ asset('bundles/ibwjobeet/css/job.css') }}" type="text/css" media="all" />
   {% endblock %}

   {% block content %}
       {% if app.request.get('token') %}
           {% include 'IbwJobeetBundle:Job:admin.html.twig' with {'job': entity} %}
       {% endif %}
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

           <h4>{% trans %}How to apply?{% endtrans %}</h4>

           <p class="how_to_apply">{{ entity.howtoapply }}</p>

           <div class="meta">
               <small>{% trans with {'%date%': entity.createdat|date('m/d/Y')} %}posted on %date%{% endtrans %}</small>
           </div>
       </div>
   {% endblock %}

src/Ibw/JobeetBundle/Resources/views/Job/new.html.twig

.. code-block:: html+jinja

   <!-- ... -->
   {% block content %}
       <h1>{% trans %}Job creation{% endtrans %}</h1>
       <!-- ... -->
           <br /> {% trans %}Whether the job can also be published on affiliate websites or not.{% endtrans %}
       <!-- ... -->
   <!-- ... -->

src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig

.. code-block:: html+jinja

   <!-- ... -->
       {% if category.morejobs %}
           <div class="more_jobs">
               {% trans with {'%count%': '<a href="' ~ path('IbwJobeetBundle_category', { 'slug': category.slug }) ~ '">' ~  category.morejobs ~ '</a>'} %}and %count% more...{% endtrans %}
           </div>
       {% endif %}
   <!-- ... -->

src/Ibw/JobeetBundle/Resources/views/Job/edit.html.twig

.. code-block:: html+jinja

   <!-- ... -->
   {% block content %}
       <h1>{% trans %}Job edit{% endtrans %}</h1>
       <!-- ... -->
           <br /> {% trans %}Whether the job can also be published on affiliate websites or not.{% endtrans %}
       <!-- ... -->
   <!-- ... -->

src/Ibw/JobeetBundle/Resources/views/Job/admin.html.twig

.. code-block:: html+jinja

   <div id="job_actions">
       <h3>Admin</h3>
       <ul>
           {% if not job.isActivated %}
               <ul>
                   <li><a href="{{ path('ibw_job_edit', { 'token': job.token }) }}">{% trans %}Edit{% endtrans %}</a></li>
                   <li>
                       <form action="{{ path('ibw_job_publish', { 'token': job.token }) }}" method="post">
                           {{ form_widget(publish_form) }}
                               <button type="submit">{% trans %}Publish{% endtrans %}</button>
                       </form>
                   </li>
               </ul>
           {% endif %}
           <li>
               <form action="{{ path('ibw_job_delete', { 'token': job.token }) }}" method="post">
                   {{ form_widget(delete_form) }}
                       <button type="submit" onclick="if(!confirm('{% trans %}Are you sure?{% endtrans %}')) { return false; }">{% trans %}Delete{% endtrans %}</button>
               </form>
           </li>
           {% if job.isActivated %}
               <li {% if job.expiresSoon %} class="expires_soon" {% endif %}>
                   {% if job.isExpired %}
                       {% trans %}Expired{% endtrans %}
                   {% else %}
                       {% trans with {'%count%':'<strong>' ~ job.getDaysBeforeExpires ~ '</strong>' } %}Expires in %count% days{% endtrans %}
                   {% endif %}

                   {% if job.expiresSoon %}
                       <form action="{{ path('ibw_job_extend', { 'token': job.token }) }}" method="post">
                           {{ form_widget(extend_form) }}
                               <button type="submit" value="Extend">{% trans %}Extend{% endtrans %}</button> {% trans %}for another 30 days{% endtrans %}
                       </form>
                   {% endif %}
               </li>
           {% else %}
               <li>
                   [{% trans with {'%url%': '<a href="' ~ url('ibw_job_preview', { 'token': job.token, 'company': job.companyslug, 'location': job.locationslug, 'position': job.positionslug }) ~ '">URL</a>'} %}Bookmark this %url% to manage this job in the future{% endtrans %}.]
               </li>
           {% endif %}
       </ul>
   </div>

src/Ibw/JobeetBundle/Resources/views/Category/show.html.twig

.. code-block:: php

   {% extends 'IbwJobeetBundle::layout.html.twig' %}

   {% block title %}
       {% trans with {'%category%': category.name} %}Jobs in the %category% category{% endtrans %}
   {% endblock %}

   {% block stylesheets %}
       {{ parent() }}
       <link rel="stylesheet" href="{{ asset('bundles/ibwjobeet/css/jobs.css') }}" type="text/css" media="all" />
   {% endblock %}

   {% block content %}
       <div class="category">
           <div class="feed">
               <a href="{{ path('IbwJobeetBundle_category', { 'slug': category.slug, '_format': 'atom' }) }}">Feed</a>
           </div>
           <h1>{{ category.name }}</h1>
       </div>

       {% include 'IbwJobeetBundle:Job:list.html.twig' with {'jobs': category.activejobs} %}

       {% if last_page > 1 %}
           <div class="pagination">
               <a href="{{ path('IbwJobeetBundle_category', { 'slug': category.slug, 'page': 1 }) }}">
                   <img src="{{ asset('bundles/ibwjobeet/images/first.png') }}" alt="First page" title="First page" />
               </a>

               <a href="{{ path('IbwJobeetBundle_category', { 'slug': category.slug, 'page': previous_page }) }}">
                   <img src="{{ asset('bundles/ibwjobeet/images/previous.png') }}" alt="Previous page" title="Previous page" />
               </a>

               {% for page in 1..last_page %}
                   {% if page == current_page %}
                       {{ page }}
                   {% else %}
                       <a href="{{ path('IbwJobeetBundle_category', { 'slug': category.slug, 'page': page }) }}">{{ page }}</a>
                   {% endif %}
               {% endfor %}

               <a href="{{ path('IbwJobeetBundle_category', { 'slug': category.slug, 'page': next_page }) }}">
                   <img src="{{ asset('bundles/ibwjobeet/images/next.png') }}" alt="Next page" title="Next page" />
               </a>

               <a href="{{ path('IbwJobeetBundle_category', { 'slug': category.slug, 'page': last_page }) }}">
                   <img src="{{ asset('bundles/ibwjobeet/images/last.png') }}" alt="Last page" title="Last page" />
               </a>
           </div>
       {% endif %}

       <div class="pagination_desc">
           {% transchoice total_jobs with {'%count%': '<strong>' ~ total_jobs ~ '</strong>'} %}
               {0} No job in this category|{1} One job in this category|]1,Inf] %count% jobs in this category
           {% endtranschoice %}
           {% if last_page > 1 %}
               - page <strong>{{ current_page }}/{{ last_page }}</strong>
           {% endif %}
       </div>
   {% endblock %}

src/Ibw/JobeetBundle/Resources/views/Affiliate/wait.html.twig

.. code-block:: html+jinja

   {% extends "IbwJobeetBundle::layout.html.twig" %}

   {% block content %}
       <div class="content">
           <h1>{% trans %}Your affiliate account has been created{% endtrans %}</h1>
           <div style="padding: 20px">
               {% trans %}Thank you!
               You will receive an email with your affiliate token
               as soon as your account will be activated.{% endtrans %}
           </div>
       </div>
   {% endblock %}

src/Ibw/JobeetBundle/Resources/views/Affiliate/affiliate_new.html.twig

.. code-block:: html+jinja

   <!-- ... -->
       <h1>{% trans %}Become an affiliate{% endtrans %}</h1>
   <!-- ... -->

| 各翻訳は、一意の id 属性を持つ ``trans-unit`` タグによって管理されます。
| これで、このファイルを編集してフランス語の翻訳を追加できます。

..
   Each translation is managed by a trans-unit tag which has a unique id attribute.
   You can now edit this file and add translations for the French language:

src/Ibw/JobeetBundle/Resources/translations/messages.fr.xlf

.. code-block:: jinja

   <?xml version="1.0"?>
   <xliff version="1.2" xmlns="urn:oasis:names:tc:xliff:document:1.2">
       <file source-language="en" datatype="plaintext" original="file.ext">
           <body>
               <trans-unit id="1">
                   <source>Jobeet - Your best job board</source>
                   <target>Jobeet - Les meilleurs offres d'emplois</target>
               </trans-unit>
               <trans-unit id="2">
                   <source>Enter some keywords (city, country, position, ...)</source>
                   <target>Entre des mots cle (ville, pays, position, ...)</target>
               </trans-unit>
               <trans-unit id="3">
                   <source>Recent viewed jobs:</source>
                   <target>Dernier emplois vus:</target>
               </trans-unit>
               <trans-unit id="4">
                   <source>About Jobeet</source>
                   <target>Apropos de Jobeet</target>
               </trans-unit>
               <trans-unit id="5">
                   <source>Become an affiliate</source>
                   <target>Devenir un affilie</target>
               </trans-unit>
               <trans-unit id="6">
                   <source>and %count% more...</source>
                   <target>et %count%
                           autres...</target>
               </trans-unit>
               <trans-unit id="7">
                   <source>Language</source>
                   <target>Langue</target>
               </trans-unit>
               <trans-unit id="8">
                   <source>Publish</source>
                   <target>Publier</target>
               </trans-unit>
               <trans-unit id="9">
                   <source>Edit</source>
                   <target>Editer</target>
               </trans-unit>
               <trans-unit id="10">
                   <source>Are you sure?</source>
                   <target>Etes-vous sur?</target>
               </trans-unit>
               <trans-unit id="11">
                   <source>Delete</source>
                   <target>Supprimer</target>
               </trans-unit>
               <trans-unit id="12">
                   <source>Extend</source>
                   <target>Prolonger</target>
               </trans-unit>
               <trans-unit id="13">
                   <source>for another 30 days</source>
                   <target>pour 30 jours supplementaires</target>
               </trans-unit>
               <trans-unit id="14">
                   <source>Bookmark this %url% to manage this job in the future</source>
                   <target>Marquer cette %url% pour gerer ce travail a l'avenir</target>
               </trans-unit>
               <trans-unit id="15">
                   <source>Whether the job can also be published on affiliate websites or not.</source>
                   <target>Si le travail peut egalement etre publie sur les sites affilies ou non.</target>
               </trans-unit>
               <trans-unit id="16">
                   <source>%company% is looking for a %position%</source>
                   <target>%company% est a la recherche d'un %position%</target>
               </trans-unit>
               <trans-unit id="17">
                   <source>How to apply?</source>
                   <target>comment appliquer?</target>
               </trans-unit>
               <trans-unit id="18">
                   <source>posted on %date%</source>
                   <target>poste en %date%</target>
               </trans-unit>
               <trans-unit id="19">
                   <source>{0} No job in this category|{1} One job in this category|]1,Inf] %count% jobs in this category</source>
                   <target>{0}Aucune annonce dans cette categorie|{1}Une annonce dans cette categorie|]1,+Inf] %count% annonces dans cette categorie</target>
               </trans-unit>
               <trans-unit id="20">
                   <source>Jobs in the %category% category</source>
                   <target>Travails dans le %category% categorie</target>
               </trans-unit>
               <trans-unit id="21">
                   <source>Your affiliate account has been created</source>
                   <target>Votre compte d'affiliation a ete cree</target>
               </trans-unit>
               <trans-unit id="22">
                   <source>Thank you!
               You will receive an email with your affiliate token
               as soon as your account will be activated.</source>
                   <target>On te remercie! Vous recevrez un email avec votre jeton d'affiliation des que votre compte sera active.</target>
               </trans-unit>
               <trans-unit id="23">
                   <source>Expires in %count% days</source>
                   <target>Expire en %count% jours</target>
               </trans-unit>
               <trans-unit id="24">
                   <source>Ask for people</source>
                   <target>Recherche des gens</target>
               </trans-unit>
               <trans-unit id="25">
                   <source>Ask for a job</source>
                   <target>Recherche d'un emploi</target>
               </trans-unit>
               <trans-unit id="26">
                   <source>Jobeet API</source>
                   <target>API Jobeet</target>
               </trans-unit>
               <trans-unit id="27">
                   <source>Job creation</source>
                   <target>Creation d'emploi</target>
               </trans-unit>
               <trans-unit id="28">
                   <source>Job edit</source>
                   <target>Edit l'emploi</target>
               </trans-unit>
               <trans-unit id="29">
                   <source>Expired</source>
                   <target>Expiré</target>
               </trans-unit>
               <trans-unit id="30">
                   <source>Full feed</source>
                   <target>Fil RSS</target>
               </trans-unit>
               <trans-unit id="31">
                   <source>Post a Job</source>
                   <target>Poste un emploi </target>
               </trans-unit>
           </body>
       </file>
   </xliff>

新しい翻訳を追加するたびに、後にキャッシュをクリアする必要があります。

.. Each time you add a new translation, you will need to clear the cache after.

.. seealso::
    *Symfony2日本語ドキュメント*

    豊富な日本語ドキュメントがありますので合わせて読み進めてみましょう。

    * `ガイドブック »翻訳  <http://docs.symfony.gr.jp/symfony2/book/translation.html>`_


.. include:: common/license.rst.inc
