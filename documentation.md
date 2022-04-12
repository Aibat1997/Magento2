**Корневая директория:** `project_directory/app/code`

#### Структура папок:
`app/code/Vendor` — название поставщика/разработчика (например: Astrio). 
`app/code/Vendor/Module_Name` — название модуля.  
`app/code/Vendor/Module_Name/etc` — содержит файлы конфигурации.

### 1. Регистрация модуля:

<h4 id="module-config">1.1 Создаем файл `Module_Name/etc/module.xml` с содержимым:</h4>

```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Module/etc/module.xsd">
    <module name="Vendor_ModuleName" setup_version="1.0.0"></module>
</config>
```

`name` - это название производителя_название модуля;  
`setup_version` - версия модуля;  

#### 1.2 Создаем файл `Module_Name/registration.php` с содержимым:

```php
<?php
\Magento\Framework\Component\ComponentRegistrar::register(
    \Magento\Framework\Component\ComponentRegistrar::MODULE,
    'Vendor_ModuleName',
    __DIR__
);
```

> Заменяем Vendor_ModuleName на нужное нам название (название производителя_название модуля).

#### 1.3 Запускаем модуль:
+ Проверяем корректно ли был создан модуль, запускаем команду:  
`php bin/magento module:status`
> Модуль должен быть в списке отключенных модулей 
+ Включим наш модуль:   
`php bin/magento module:enable Vendor_Module`
+ Обновим структуры базы данных:    
`php bin/magento setup:upgrade`

### 2. Роутинг:
Cсылка делится на три составных элемента:  
`http://magento2.com/route_name/controller/action`  

`route_name` - название роута (в **routes.xml**);  
`controller` - папка в каталоге **Controller**;   
`action` - php класс с методом **execute**;   

#### 2.1 Регистрация роута:  

##### 2.1.1 Создаем файл `Module_Name/etc/frontend/routes.xml` с содержимым:

```xml
<?xml version="1.0" ?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:App/etc/routes.xsd">
    <router id="standard">
        <route frontName="helloworld" id="helloworld">
            <module name="Vendor_ModuleName"/>
        </route>
    </router>
</config>
```

router `id` -     
route `frontName` - название роута в адресной строке;     
route `id` - идентификатор роута(применяется в [шаблоне](#321-создадим-layout-файл-в-modulenameviewfrontendlayouthelloworldindexindexxml-с-содержимым));   
module `name` - название модуля;

### 3. MVC

#### 3.1 Model

В Magento2 модель разделена на три части, это:
+ **ResourceModel** - для совершения действий с базой данных (CRUD);
+ **Model** - содержит бизнес-логику модели;
+ **Collection** - для фильтрования и сортировки данных модели;

<h5 id="resource-model">3.1.1 Создадим модель ресурсов в `ModuleName/Model/ResourceModel/ResourceModelName.php` с содержимым:</h5>

```php
<?php

namespace Vendor\ModuleName\Model\ResourceModel;

class ResourceModelName extends \Magento\Framework\Model\ResourceModel\Db\AbstractDb
{
	
	public function __construct(
		\Magento\Framework\Model\ResourceModel\Db\Context $context
	)
	{
		parent::__construct($context);
	}
	
	protected function _construct()
	{
		$this->_init('vendor_modulename_tablename', 'pk_id');
	}
	
}
```

> Все фактические операции с базой данных выполняются моделью ресурсов. Каждая модель должна иметь модель ресурсов.

<h5 id="model">3.1.2 Создадим модель в `ModuleName/Model/ModelName.php` с содержимым:</h5>

```php
<?php

namespace Vendor\ModuleName\Model;

class ModelName extends \Magento\Framework\Model\AbstractModel implements \Magento\Framework\DataObject\IdentityInterface
{
	const CACHE_TAG = 'vendor_modulename_tablename';

	protected $_cacheTag = 'vendor_modulename_tablename';

	protected $_eventPrefix = 'vendor_modulename_tablename';

	protected function _construct()
	{
		$this->_init('Vendor\ModuleName\Model\ResourceModel\ResourceModelName');
	}

	public function getIdentities()
	{
		return [self::CACHE_TAG . '_' . $this->getId()];
	}

	public function getDefaultValues()
	{
		$values = [];

		return $values;
	}
}
```

>  метод _init() определяет [модель ресурсов](#resource-model), которая фактически будет извлекать информацию из базы данных.

##### 3.1.3 Создадим модель коллекции в `ModuleName/Model/ResourceModel/ModelName/Collection.php` с содержимым:

```php
<?php

namespace Vendor\ModuleName\Model\ResourceModel\ModelName;

class Collection extends \Magento\Framework\Model\ResourceModel\Db\Collection\AbstractCollection
{
	protected $_idFieldName = 'pk_id';
	protected $_eventPrefix = 'vendor_modulename_tablename_collection';
	protected $_eventObject = 'modelname_collection';

	/**
	 * Define resource model
	 *
	 * @return void
	 */
	protected function _construct()
	{
		$this->_init('Vendor\ModuleName\Model\ModelName', 'Vendor\ModuleName\Model\ResourceModel\ResourceModelName');
	}

}
```

>  метод _init() определяет [модель](#model) и [модель ресурсов](#resource-model)

#### 3.2 View

Состоит из трех компонентов (подробнее [здесь](https://devdocs.magento.com/guides/v2.3/frontend-dev-guide/layouts/layout-overview.html))
+ Layouts - описывает структуру веб-страницы;
+ Containers - секции для заполнения контентом;
+ Blocks - элементы пользовательского интерфейса на странице. 

##### 3.2.1 Создадим layout файл в `ModuleName/view/frontend/layout/helloworld_index_index.xml` с содержимым:

> структура расоложения шаблона `module_name`/view/`area`/layout, где:
> + `area` - может быть **frontend**(область пользователя) или **adminhtml**(область админ панели);  

> название layout-a состоит из `routerId`_`controllerName`_layoutName.xml, где:
> + `routerId` - [id](#21-регистрация-роута) роута;
> + `controllerName` - название контроллера;

```xml
<?xml version="1.0"?>
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" layout="1column" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <referenceContainer name="content">
        <block class="Vendor\ModuleName\Block\BlockName" name="helloworld_index_index" template="Vendor_ModuleName::index.phtml" />
    </referenceContainer>
</page>
```

##### 3.2.2 Создадим блок файл в `ModuleName/Block/BlockName.php` с содержимым: 

```php
<?php

namespace Mageplaza\HelloWorld\Block;

class BlockName extends \Magento\Framework\View\Element\Template
{

}
```

##### 3.2.3 Создадим файл шаблона в `ModuleName/view/frontend/templates/index.phtml` с содержимым: 

```html
<h2>Welcome to Mageplaza.com</h2>
```

#### 3.3 Controller

##### 3.3.1 Создадим контроллер `ModuleName/Controller/ControllerName/Action.php` с содержимым:

```php
<?php

namespace Vendor\ModuleName\Controller\ControllerName;

class Action extends \Magento\Framework\App\Action\Action
{
    protected $_pageFactory; //для шаблона
	protected $_modelnameFactory; //для модели

    public function __construct(
        \Magento\Framework\App\Action\Context $context,
        \Magento\Framework\View\Result\PageFactory $pageFactory,
		\Vendor\ModuleName\Model\ModelNameFactory $modelnameFactory
    ) {
        $this->_pageFactory = $pageFactory;
		$this->_modelnameFactory = $modelnameFactory;
        return parent::__construct($context);
    }

    public function execute()
    {
        //ваш код

		//для модели
		//$model = $this->_modelnameFactory->create();
		//$collection = $model->getCollection();

        //return $this->_pageFactory->create(); //для шаблона (view)
    }
}
```

### 4. Действия с Базой Данных

#### 4.1 Создание таблицы 

+ В файле `ModuleName/Setup/InstallSchema.php` распишем структуру таблицы: 

```php
<?php
namespace Vendor\ModuleName\Setup;

class InstallSchema implements \Magento\Framework\Setup\InstallSchemaInterface
{

	public function install(\Magento\Framework\Setup\SchemaSetupInterface $setup, \Magento\Framework\Setup\ModuleContextInterface $context)
	{
		$installer = $setup;
		$installer->startSetup();
		if (!$installer->tableExists('vendor_modulename_tablename')) {
			$table = $installer->getConnection()->newTable(
				$installer->getTable('vendor_modulename_tablename')
			)
				->addColumn(
					'pk_id',
					\Magento\Framework\DB\Ddl\Table::TYPE_INTEGER,
					null,
					[
						'identity' => true,
						'nullable' => false,
						'primary'  => true,
						'unsigned' => true,
					],
					'PK ID'
				)
                ->addColumn(
					'name',
					\Magento\Framework\DB\Ddl\Table::TYPE_TEXT,
					255,
					['nullable => false'],
					'Name'
				);

			$installer->getConnection()->createTable($table);
		}

		$installer->endSetup();
	}
}
```

#### 4.2 Изменение структуры таблицы 

+ В файле `ModuleName/Setup/UpgradeSchema.php` распишем структуру таблицы: 

```php
<?php
namespace Vendor\ModuleName\Setup;

use Magento\Framework\Setup\UpgradeSchemaInterface;
use Magento\Framework\Setup\SchemaSetupInterface;
use Magento\Framework\Setup\ModuleContextInterface;

class UpgradeSchema implements UpgradeSchemaInterface
{
	public function upgrade( SchemaSetupInterface $setup, ModuleContextInterface $context ) {
		$installer = $setup;

		$installer->startSetup();

		if(version_compare($context->getVersion(), '1.1.0', '<')) {
			if (!$installer->tableExists('vendor_modulename_tablename')) {
				$table = $installer->getConnection()->newTable(
					$installer->getTable('vendor_modulename_tablename')
				)
					->addColumn(
						'pk_id',
						\Magento\Framework\DB\Ddl\Table::TYPE_INTEGER,
						null,
						[
							'identity' => true,
							'nullable' => false,
							'primary'  => true,
							'unsigned' => true,
						],
						'PK ID'
					)
					->addColumn(
						'name',
						\Magento\Framework\DB\Ddl\Table::TYPE_TEXT,
						255,
						['nullable => false'],
						'Name'
					);

				$installer->getConnection()->createTable($table);
			}
		}

		$installer->endSetup();
	}
}
```

> тут идет сравнение c прошлой [версией модуля](#module-config)

>  При каждом обновлений структуры базы данных нужно запускать команду `php bin/magento setup:upgrade`
