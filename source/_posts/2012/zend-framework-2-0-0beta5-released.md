---
title: "Zend Framework 2 beta5发布，增加i18n支持，更新Form功能"
date: "2012-07-09 10:45:26"
slug: "zend-framework-2-0-0beta5-released"
published: true
author: "AlloVince"
legacy_id: 152
comment_status: "open"
comments: true
gitalk_id: "POST_152"
tags: []
---
如果不出意外的话，这应该是ZF2的最后一个Beta版，之后就会进入RC阶段。

本次的[beta5更新](/pages/zend-framework-2-0-0beta5-released "http://avnpc.com/pages/zend-framework-2-0-0beta5-released")主要的变动有两个，一是增加了i18n国际化支持，二是对ZF2 Form增加了一个名为Annotations（表单注释）的新功能。

Zf2的I18N并没有特别大的改进，目前只支持[Gettext](http://zh.wikipedia.org/wiki/Gettext "http://zh.wikipedia.org/wiki/Gettext")或者php array两种形式的翻译库文件。然后在实际使用时，可以有两种方法实施i18n，一种是指定一个特定的翻译库文件：

```php
$translator = Zend\I18n\Translator\Translator::factory(array(
    'locale' => 'zh_CN',
    'translation_files' => array(
        array(
            'type' => 'phparray',
            'filename' =>  __DIR__ . '/_files/translation-zh_CN.php',
        )
    ),
));
echo $translator->translate('Dream');
```

另一种是根据语言名称切换不同的翻译库文件

```php
$translator = Zend\I18n\Translator\Translator::factory(array(
    'locale' => 'zh_CN',
    'translation_patterns' => array(
        array(
            'type' => 'phparray',
            'base_dir' => __DIR__ . '/_files',
            'pattern' => 'translation-%s.php'
        )
    ),
));
echo $translator->translate('Dream');
```

当本地语言为en时，上例会自动寻找translation-en.php的文件。

ZF2的Form Annotations则是一个有意思的功能，一般来说表单都需要定义表单元素的属性，包括输入框的类型，名称，说明，验证规则等等。在php中经常使用数组来定义，比如使用常规的ZF2 Form，我们可以在Form中这样添加一个元素：

```php
class MyForm extends \Zend\Form\Form
{
 
    protected function init()
    {
 
        $factory = $this->getFormFactory();
        $this->add($factory->createElement(array(
            'name' => 'title',
            'attributes' => array(
                'label' => 'Title:',
                'type'  => 'text',
            ),
        )));
        $this->setInputFilter($factory->createInputFilter(array(
            'title' =>     array(
                'name' => 'title',
                'required' => true,
                'validators' => array(
                    array(
                        'name' => 'StringLength',
                        'options' => array(
                            'encoding' => 'UTF-8',
                            'max' => 100,
                        ),
                    ),
                ),
            ),
        )));
    }
}
```

根据数组定义，我们增加了一个text输入框，并且加上了输入验证，要求最大输入长度为100个字符。

而在Form Annotations中，我们可以把对表单元素的定义放在**代码注释**里，是的你没有看错，ZF2可以通过读取代码注释获得表单元素的信息。所以上例可以写成：

```php
use Zend\Form\Annotation;
/**
 * @Annotation\Hydrator("Zend\Stdlib\Hydrator\ObjectProperty")
 * @Annotation\Name("post")
 */
class Post
{
    /**
     * @Annotation\Attributes({"type":"text" })
     * @Annotation\Validator({"type":"StringLength","options":{"encoding":"UTF-8","max":"100"}})
     * @Annotation\Options({"label":"Title:"})
     */
    public $title;
}
```

之后通过AnnotationBuilder构建一个表单

```php
use Zend\Form\Annotation\AnnotationBuilder;
 
$post    = new Post();
$builder = new AnnotationBuilder();
$form    = $builder->createForm($post);
 
$form->bind($post);
echo $form->username;
```

这个神奇的功能是通过ZF2的反射Zend\Code\Reflection实现的，有兴趣的话不妨看看源代码的实现方法。

个人而言，在常规的Array与Zend\Form\Annotation两种方法中，还是更倾向使用Array方法。因为在实际需求中，我们经常会遇到这样的场景：我们需要一个表单创建一篇文章，文章标题是唯一的。同样的表单还会用作编辑表单，但是在编辑表单时，文章标题的验证规则就有了微小的变化，验证标题的唯一需要排除当前文章本身。

再比如我们创建文章时可能需要填写作者姓名，但是文章发表之后作者姓名就无需再次填写，表单本身包含的元素也发生了微小的变化。

所以对于Form的使用，我们最大的需求应该是一个可以被复写，灵活修改的元素配置。在这一点上，数组有更大的优势。

在EvaEngine中，针对Form的这一特性，使用Array作为元素配置，并且引入了子表单的概念，让Form的使用更加灵活的一些，欢迎提出意见。请参看[EvaEngine的Form实现](https://github.com/AlloVince/eva-engine/blob/master/vendor/Eva/Form/Form.php "https://github.com/AlloVince/eva-engine/blob/master/vendor/Eva/Form/Form.php")。
