# Tasks for QA:

- There are some failing specs in the `features` directory.
Your task is to fix them by writing factories and using them in your test.
- For each task I've prepared some tips.
- Please remember, that all capitilized values should be replaced by qa with his own code.

## Task 1 - writing first factory
In this task you are going to get familiar with basic FactoryGirl usage.
You will need to write a factory for the `Bird` model, which should be placed in `spec/factories/birds.rb` file.
I've already prepared some basic structure in this file, let' take a look at this:
```
FactoryGirl.define do
  factory :bird do

  end
end

```
The very first line tells facorygirl that we will want to create factory for new model, specified further
as `factory :bird`. By default name of your factory should match name of the model
(this behaviour can be overwrittn, but let's skip it for now :) ). To create our first working factory, we need to know which fields
does our model have. We can find those informations in `db/schema.rb` file.
Every model in this project has corresponding database table, represented in `schema.rb`, f.ex.
for he `Bird` model we will have corresponding `birds` table with specified list of fields:
```
create_table "birds", force: :cascade do |t|
  t.datetime "created_at",  null: false
  t.datetime "updated_at",  null: false
  t.string   "name"
  t.text     "description"
  t.string   "latin_name"
end
```
We can see 5 fields listed in schema for birds table:
- created_at
- updated_at
- name
- description
- latin_name
The first two fields are automatically generated by rails and we do not need to specify them in our factory,
yet we should add the last three. You can add the to your factory by writing new line with field name
and default value, like this:
```
FactoryGirl.define do
  factory :bird do
    name "Rudzik"
  end
end
```

Use the example above and add `description` and `latin_name` to your factory...

After finishing this factory, go to the console and type in `rails c test`. This command will load rails console in test environment,
allowing you to access FactoryGirl commands.
Now, type the command below to create new Bird object using your very first factory:
`FactoryGirl.create(:bird)`
Viola! You've just created your first object using FactoryGirl!
But... wait, when you try to run the same command once again, you will see an error:
`Validation failed: Name has already been taken, Latin name has already been taken`
So, what's wrong? Take a look at the `app/models/bird.rb` file, where you should see the following lines:
```
validates :name, presence: true, uniqueness: true
validates :latin_name, presence: true, uniqueness: true
validates :description, presence: true
```
It appears that there is one more place where you should take a look when writing your factory,
which is the model file. Developers pretty often lavy validations in two places:
database (represented by `schema.rb`) and in model files, usually placed in `app/models` directory.

How to avoid such an errors when writing specs? We will learn this in next tasks.

## Task 2 - using FFaker gem.
One of the most useful tools when it comes to writing factories is [FFaker](https://github.com/ffaker/ffaker).
Before taking step further, please see its readme - especially the "Usage" section.

What is this whole FFaker about? - you may ask... FFaker helps you to create random data for fields in your factories.
If you want to randomize the data for you tests, instead of typing hardcoded data into factory, like this:
```
FactoryGirl.define do
  factory :bird do
    name "Rudzik"
  end
end
```
You should use FFaker. How to use it? It is pretty simple, just look at the example:
```
FactoryGirl.define do
  factory :bird do
    name { FFaker::Name.first_name }
  end
end
```
and that's it! Now every time you will try to build or create an object using the factory above,
FFaker will do its best to randomize the data. Of course it is not perfect and it is
possible that when creating huge amount of objects, some duplicates will occure.

To make completely sure that we won't get any duplicates, we should use method called `sequence` - we will cover this topic in next few tasks.
For now, please try to write your factory for the Bird and Country model using FFaker.

## Task 3 - belongs_to associations
In this task we will try to write factory for the next model, called `Region`.
While browsing for informations about this model in `schema.rb`, you may spot something
pretty unusual: `regions` table has some pretty weird field, called
`country_id`... What is it responsible for and how to fill this field in you factory?

As `app/models/region.rb` file tells this model `belongs_to :country`. `country_id` holds the information
about which country given region belongs to. How do we fill this in? Take a look at the example below:
```
FactoryGirl.define do
  factory :region do
    country
  end
end
```
Writing the associated model name in the factory is all we need! FactoryGirl will figure it out and create associated country, but
this requires the `Country` factory, which you should have written in the previous task :)

You can read more about associations in FactoryGirl [here](http://www.rubydoc.info/gems/factory_girl/file/GETTING_STARTED.md#Associations).

## Task 4 - has_many association
As you may already know, there are more association types in rails. One of the most commonly used is
`has_many`. In this project you can spot it in the country model, which `has_many :regions`.
How to deal with this type of associations? We will need to learn one new technique: transients.

Now, when we have the region factory defined, we can use transients to create factory for a country with 3 regions.
The example below will guide you in this task:
```
FactoryGirl.define do
  factory :country do
    <-- some code from task 2 here -->

    factory :country_with_regions do
      transient do
        regions_count 3
      end

      after(:create) do |country, evaluator|
        create_list(:region, evaluator.regions_count, country: country)
      end
    end
  end
end
```
Ok, here is some black magic which needs further explanation:
* what is this `after(:create)` block doing?
  * After creating Country object it takes the country and evaluator to
    create the list of regions with count equal to `evaluator.regions_count` (which is 3)
    and assigns them to this brand new country object.
* how can we use this in `rails c test` console?
  * `FactoryGirl.create(:country_with_regions)``
  * `FactoryGirl.create(:country_with_regions, regions_count: 5)`
* how can we use it in rspec test?
  * `let!(:country_with_some_regions) { create(:country_with_regions) }`
  * `let!(:country_with_more_regions) { FactoryGirl.create(:country_with_regions, regions_count: 5) }`

The solution above will work perfectly for our needs, yet there are some things which could be done better,
like using traits in place of nested factories (in exmple above `:country_with_regions`
is nested inside `:country`). Traits not only make cod more readable but can be chained together
to easily create mor complex factories. If we would like to create factory for country with many
regions and many environmental_laws - which will be your task, we won't need to define 3 separate
factories for for each type of country (with environmental_laws, with regions and with both of them), the onyl thing we will need
are mixed traits.