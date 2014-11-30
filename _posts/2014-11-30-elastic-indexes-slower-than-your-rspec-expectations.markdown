---
layout: post
title:  "Elasticsearch indexes are slower then your rspec expectations"
date:   2014-11-30 23:14:00
---

If you want to play nice with rspec and elasticsearch you'd need to wait for elastic indexes
to actually be created before you do any expectations. In other case expect them to be <span class='red'>RED</span>.

Guys from [pivotal](http://pivotallabs.com/rspec-elasticsearchruby-elasticsearchmodel/) recommend to use `sleep`.
But wait, `sleep` is just not the ruby way :)

Go hardcore, use `__elasticsearch__.refresh_index!`, here's a snippet from my tests:

{% highlight ruby %}
describe '.predict_answer', elasticsearch: true do
  subject { LegalCase.predict_answer(question).records.to_a }

  context 'find by question text' do
    let(:question) { 'case 1' }
    let!(:case_1)  { create(:legal_case, question: 'case 1') }
    let!(:case_2)  { create(:legal_case, question: 'something not related') }

    it { is_expected.to eq([ case_1 ]) }
  end
end
{% endhighlight %}

Wait! - You say. Where do you wait for indexes?
I have created a helper file for rspec:

{% highlight ruby %}
# Warning, this is for elasticsearch usage only
module RspecOverrides

  def is_expected
    described_class.__elasticsearch__.refresh_index!
    super
  end

end
{% endhighlight %}

And I load this helper only for `elasticsearch: true` specs.

Like this:
{% highlight ruby %}
...
config.include FactoryGirl::Syntax::Methods
config.include RspecOverrides, elasticsearch: true # only in elastic specs
...
{% endhighlight %}

That's all folks. Keep the code clean and tidy.