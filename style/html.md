# HTML Styleguide

1. 4-spaces for indentation

1. Use double quotes `"` for attributes

1. If your IDE is able, place the second and all subsequent attributes on a new line indented 4 spaces from the opening brace
    + this is currently _lightly_ enforced via a [.jsbeautifyrc](/style/.jsbeautifyrc) file
    ```html
    <div class="btn btn-primary"
        id="hello-world"
        ng-if="customer.isLoggedIn">
        <img ng-src="{{customer.avatar}}"
            alt="{{customer.name" />
        <a class="customer-edit"
            ng-click="customer.edit()">
            <i class="fa fa-gear"></i>
        </a>
    </div>
    ```

1. css names are limited to the following characters
    + a-z; lowercased
    + 0-9; but never at the beginning of the class name
    + the special characters; `_`, `-`