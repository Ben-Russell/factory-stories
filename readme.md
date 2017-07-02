## Laravel Model Factory Stories ##
This package allows You to create complex model factories in separated classes reusable among Your tests classes.

inspired by the [Model Factories podcast](http://twentypercent.fm/model-factories) on [twentypercent.fm](http://twentypercent.fm)

## Installing ##
    composer require jeffochoa/factory-stories

Edit your app.php file

    FactoryStories\Providers\StoryFactoryServiceProvider::class

## The problem ##
Let's say You have to do some tests over articles created with certain conditions like:

    // Active articles, from Active Users, with tags attached
You may end creating something like

    $user = factory(User::class)->states('active');
    $tags = factory(Taxonomy::class, 3)->states('tag')->create();
    $article = factory(Article::class)->create([
        user_id => $user->id
    ]);
    $article->tags()->attach($tags->pluck('id')->toArray());
... or something like that.

Of course, You allways can extract this to it's own method in a helper class, but sometimes You may want to have each of this kind of "stories" in it's own class, even  more when you need to add some extra methods to generate more complex data.

## Creating new Factory Stories ##

After install You'll have access to a a new artisan command

    $ php artisan make:factory-story SomeClassName
After run this command You should see the new file under the Tests\Stories directory

    <?php

    namespace Tests\Stories;

    use App\Models\User;
    use FactoryStories\Contracts\FactoryStoryContract;

    class TestStory implements FactoryStoryContract
    {
        public function handle($params = [])
        {
            // here you can add your complex model factories with their relationships
            return factory(User::class)->create();
        }

        // and You can add custom methods if You need to
    }

## Using Factory Stories ##
Consider this UserTestClass.php

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ManageUsersTest extends TestCase
    {
        use DatabaseMigrations;

        /** @test **/
        public function your_test_method()
        {
             //
        }

    }

You just need to use the story() helper and call the create() method on it

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ManageUsersTest extends TestCase
    {
        use DatabaseMigrations;

        /** @test **/
        public function your_test_method()
        {
             $article = story(\Tests\Stories\ActiveUserArticleWithTags::class, 5)
                 ->create();
        }
    }
This will return a collection of 5 items  with the object that You return in the handle() method of the custom story class.