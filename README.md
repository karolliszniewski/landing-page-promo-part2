###Create the Configuration XML File

#Create the system.xml File in 'app/code/Vendor/Module/etc/adminhtml/system.xml'

```xml
<?xml version="1.0"?>
<!-- app/code/LandingPage/Form/etc/adminhtml/system.xml -->
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Config/etc/system_file.xsd">
    <system>
        <section id="landingpage_form" translate="label" sortOrder="10" showInDefault="1" showInWebsite="1" showInStore="1">
            <label>LandingPage Form Configuration</label>
            <tab>general</tab>
            <resource>LandingPage_Form::config</resource>
            <group id="general" translate="label" sortOrder="10" showInDefault="1" showInWebsite="1" showInStore="1">
                <label>General Settings</label>
                <field id="enable_module" translate="label comment" type="select" sortOrder="10" showInDefault="1" showInWebsite="1" showInStore="1">
                    <label>Enable Module</label>
                    <comment>Choose whether to enable or disable the module.</comment>
                    <source_model>Magento\Config\Model\Config\Source\Yesno</source_model>
     
                </field>
            </group>
        </section>
    </system>
</config>
```

setting should be here :
![368539070-2ea82d7d-9c0a-456c-a053-0c772bf83b32](https://github.com/user-attachments/assets/e14eb300-83b5-4129-b8cb-09101c774e6c)


Next step use helper:
create 'app/code/LandingPage/Form/Helper/Data.php'

Data.php content:

```php
<?php
namespace LandingPage\Form\Helper;

use Magento\Framework\App\Helper\AbstractHelper;
use Magento\Framework\App\Helper\Context;
use Magento\Framework\App\Config\ScopeConfigInterface;
use Magento\Store\Model\ScopeInterface;

class Data extends AbstractHelper
{
    const XML_PATH_MODULE_ENABLED = 'landingpage_form/general/enable_module';

    protected $scopeConfig;

    public function __construct(
        Context $context,
        ScopeConfigInterface $scopeConfig
    ) {
        parent::__construct($context);
        $this->scopeConfig = $scopeConfig;
    }

    /**
     * Check if the module is enabled
     *
     * @return bool
     */
    public function isModuleEnabled()
    {
        return $this->scopeConfig->isSetFlag(
            self::XML_PATH_MODULE_ENABLED,
            ScopeInterface::SCOPE_STORE
        );
    }
}
```

next use helper in block:
```php
<?php
namespace LandingPage\Form\Block;

use Magento\Framework\View\Element\Template;
use Magento\Customer\Model\Session as CustomerSession;
use Magento\Customer\Model\Url as CustomerUrl;
use LandingPage\Form\Helper\Data as FormHelper;

class Index extends Template
{
    protected $customerSession;
    protected $customerUrl;
    protected $formHelper;

    /**
     * Construct
     *
     * @param \Magento\Framework\View\Element\Template\Context $context
     * @param CustomerSession $customerSession
     * @param CustomerUrl $customerUrl
     * @param FormHelper $formHelper
     * @param array $data
     */
    public function __construct(
        \Magento\Framework\View\Element\Template\Context $context,
        CustomerSession $customerSession,
        CustomerUrl $customerUrl,
        FormHelper $formHelper,
        array $data = []
    ) {
        parent::__construct($context, $data);
        $this->customerSession = $customerSession;
        $this->customerUrl = $customerUrl;
        $this->formHelper = $formHelper;
    }

    public function getCustomerName()
    {
        $customer = $this->customerSession->getCustomer();
        $name = $customer->getName();

        return $name;
    }

    public function getCustomerEmail()
    {
        $customer = $this->customerSession->getCustomer();
        $email = $customer->getEmail();

        return $email;
    }

    public function getLoginUrl()
    {
        return $this->getUrl('customer/account/login');
    }

    /**
     * Check if the module is enabled
     *
     * @return bool
     */
    public function isModuleEnabled()
    {
        return $this->formHelper->isModuleEnabled();
    }
}
```

#This also should only affect the en_gb site.

#Switch Store

edit file 'pub/.htaccess'

Add this to the top:
```bash
SetEnvIf Host audio-technica.awaredev.site MAGE_RUN_CODE=en_gb
SetEnvIf Host audio-technica.awaredev.site MAGE_RUN_TYPE=store
```



### Updating Core Configuration

1. **Check the `core_config_data` Table:**
   - Search for the following keys:
     - `web`
     - `web/secure/base_url`
     - `web/unsecure/base_url`

2. **Update the URLs:**
   - For example, change:
     - `https://staging.audio-technica.com/de-de/`
   - To:
     - `https://audio-technica.awaredev.site/`

3. **Remove Unnecessary Entries:**
   - Delete the key:
     - `web/cookie/cookie_domain`
    
#Create child template 

put inside block another block (it wil make child)
```xml
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>
        <referenceContainer name="content">
            <block class="LandingPage\Form\Block\Index" name="form_block" template="LandingPage_Form::content.phtml">
                <block class="LandingPage\Form\Block\Index" name="form_block_child_disabled" template="LandingPage_Form::disabled.phtml"/>
                <block class="LandingPage\Form\Block\Index" name="form_block_child_session" template="LandingPage_Form::session.phtml"/>
            </block>   
        </referenceContainer>
    </body>
</page>
```

inside 'content.phtml' you can call child block like this :

```php
    $block->getChildHtml('form_block_child_session');
```

#The data should be collected and saved to the table:

create 'app/code/LandingPage/Form/etc/db_schema.xml'

```xml
<?xml version="1.0"?>
<schema xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:framework:Setup/Declaration/Schema/etc/schema.xsd">

    <table name="landingpage_form" resource="default" engine="innodb" comment="Landing Page Form Table">
        <column xsi:type="int" name="id" nullable="false" identity="true" unsigned="true" comment="Form ID"/>
        <column xsi:type="int" name="customer_id" nullable="false" comment="Customer ID"/>
        <column xsi:type="varchar" name="comment" nullable="true" length="255" comment="Comment"/>
        <constraint xsi:type="primary" referenceId="PRIMARY">
            <column name="id"/>
        </constraint>
    </table>

</schema>
```

run

```bash
bin/magento setup:upgrade
```


create 'app/code/LandingPage/Form/Model/FormData.php'
```php
<?php
// send FormData to AbstractModel
namespace LandingPage\Form\Model;

use Magento\Framework\Model\AbstractModel;


class FormData extends AbstractModel
{
    protected function _construct()
    {
        $this->_init(\LandingPage\Form\Model\ResourceModel\FormData::class);
    }
}
```

create 'app/code/LandingPage/Form/Model/ResourceModel/FormData'

```php
<?php
namespace LandingPage\Form\Model\ResourceModel;

use Magento\Framework\Model\ResourceModel\Db\AbstractDB;

class FormData extends AbstractDb{
    protected function _contruct(){
        $this->_int('landingpage_form', 'id');
    }
}
```

create 'app/code/LandingPage/Form/Model/ResourceModel/FormData/Collection.php'

```php
<?php
namespace LandingPage\Form\Model\ResourceModel\FormData;

use Magento\Framework\Model\ResourceModel\Db\Collection\AbstractCollection;
use LandingPage\Form\Model\FormData;
use LandingPage\Form\Model\ResourceModel\FormData as FormDataResource;

class Collection extends AbstractCollection{
    protected function _construct(){
        $this->_int(FormData::class,FormDataResource::class);
    }
}
```

### Updated File: `app/code/LandingPage/Form/view/frontend/templates/form/form.phtml`

**Changes:**
- Added the missing field: `comment`
- The form now submits to the block method: `block->getUrl('landingpage/index/post')` using the `POST` method
- Each form requires a unique token, so a `formKey` has been added for security

```phtml
<div style="width:100%;display:flex;justify-content:center;align-items:center;" class="account_container-inner">
    <form style="width:50%;" action="<?= $block->getUrl('landingpage/index/post') ?>" class="form" method="post" id="user-info-form">
        <?= $block->getBlockHtml('formkey') ?>
        <fieldset class="fieldset">
            <div class="field">
                <label class="label label--required" for="username"><span><?= __("Name")?></span></label>
                <input name="username" class="input" required id="username" type="text" title="Username" value="<?=$block->getCustomerName()?>" disabled require>
            </div>      
             
            <div class="field">
            <label class="label label--required" for="username"><span><?= __("Email")?></span></label>
                <input name="email" class="input" required id="email" type="email" title="Email" value="<?=$block->getCustomerEmail()?>" disabled>
            </div>
            <label class="label" for="username"><span><?= __("Comment")?></span></label>
            <div class="field">
            
                <textarea name="comment" class="input" id="comment" title="Comment" > </textarea>
            </div>
            <!-- Hidden fields to send username and email -->
            <input type="hidden" name="username" value="<?=$block->getCustomerName()?>">
            <input type="hidden" name="email" value="<?=$block->getCustomerEmail()?>">

            <div class="actions">
                <button type="button" class="action-primary" name="add-to-cart">
                    <span><?= __("Add to Cart") ?></span>    
                </button>
                <button class="action-secondary" type="submit">
                    <span><?= __("Read More") ?></span>
                </button>
            </div>
        </fieldset>
    </form>
</div>
```


create 'app/code/LandingPage/Form/Controller/Index/Post.php'

```php
<?php
namespace LandingPage\Form\Controller\Index;

use Magento\Framework\App\Action\Context;
use Magento\Framework\App\Action\Action;
use Magento\Framework\Controller\ResultFactory;
use Psr\Log\LoggerInterface;
// defined in 'app/code/LandingPage/Form/Model/FormData.php'
use LandingPage\Form\Model\FormDataFactory;
use Magento\Customer\Api\CustomerRepositoryInterface;

class Post extends Action
{
    protected $formDataFactory;
    protected $logger;

    /**
     * Constructor
     * 
     * @param Context $context
     * @param FormDataFactory $formDataFactory
     * @param LoggerInterface $logger
     */
    public function __construct(
        Context $context,
        FormDataFactory $formDataFactory,
        CustomerRepositoryInterface $customerRepository,
        LoggerInterface $logger
    ) {
        parent::__construct($context);
        $this->formDataFactory = $formDataFactory;
        $this->customerRepository = $customerRepository;
        $this->logger = $logger;
    }


    public function execute()
    {
        
        $postData = $this->getRequest()->getPostValue();

        
        if (!empty($postData)) {

            $customer = $this->getCustomer();

            $customer_id = $customer->getId();
           // Initialize the model
           $formData = $this->formDataFactory->create();

           //setData is from AbstractModel
           $formData->setData([
            'customer_id' => $customer_id,
            'comment' => $postData['comment']
           ]
           );

           // Save data to the database

           try{
            $formData->save();

            
           } catch (\Exception $e) {
            dd($e->getMessage());
           }

        } 

        // Optional: further processing can go here after debugging.
    }

    protected function getCustomer()
    {
        try{
            return $this->customerRepository->get($postData['email']);
        } catch (\Exception $e){
            return false;
        }
    }
}
```


now lets change $formData->save(); to interface

create interface 'app/code/LandingPage/Form/Api/FormDataInterface.php'

```php
<?php
namespace LandingPage\Form\Api;

use LandingPage\Form\Api\Data\FormDataInterface;

interface FormDataRepositoryInterface
{
    /**
     * Save form data.
     *
     * @param FormDataInterface $formData
     * @return FormDataInterface
     */
    public function save(FormDataInterface $formData);

    /**
     * Get form data by ID.
     *
     * @param int $id
     * @return FormDataInterface
     */
    public function getById($id);

    /**
     * Delete form data.
     *
     * @param FormDataInterface $formData
     * @return bool
     */
    public function delete(FormDataInterface $formData);
}
```


implement interface 'app/code/LandingPage/Form/Model/FormDataRepository.php'

```php
<?php
namespace LandingPage\Form\Model;

use LandingPage\Form\Api\FormDataRepositoryInterface;
use LandingPage\Form\Api\Data\FormDataInterface;
use LandingPage\Form\Model\ResourceModel\FormData as FormDataResource;
use Magento\Framework\Exception\CouldNotSaveException;

class FormDataRepository implements FormDataRepositoryInterface
{
    protected $resource;

    public function __construct(FormDataResource $resource)
    {
        $this->resource = $resource;
    }

    public function save(FormDataInterface $formData)
    {
        try {
            $this->resource->save($formData);
        } catch (\Exception $exception) {
            throw new CouldNotSaveException(__($exception->getMessage()));
        }
        return $formData;
    }

    public function getById($id)
    {
        $formData = $this->formDataFactory->create();
        $this->resource->load($formData, $id);
        return $formData;
    }

    public function delete(FormDataInterface $formData)
    {
        try {
            $this->resource->delete($formData);
        } catch (\Exception $exception) {
            throw new CouldNotDeleteException(__($exception->getMessage()));
        }
        return true;
    }
}
```

create 'app/code/LandingPage/Form/etc/di.xml'

```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <preference for="LandingPage\Form\Api\FormDataRepositoryInterface" type="LandingPage\Form\Model\FormDataRepository" />
</config>
```








