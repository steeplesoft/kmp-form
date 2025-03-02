
> This is a Kotlin Multiplatform compatible fork of https://github.com/benjamin-luescher/compose-form
> To make the separation clean, I have change the project name and package.

# Android KMP Form Library
![https://jitpack.io/#benjamin-luescher/compose-form](https://jitpack.io/v/benjamin-luescher/compose-form.svg)

This library provides an easy-to-use and customizable solution for building forms in Kotlin Multiplatform. It includes form fields such as text input, pickers, checkbox, and more, with built-in validators to ensure accurate user input. Data binding is also supported, making it easy to work with form data in your code.

Whether you're building a complex registration form or a simple feedback form, this library has you covered.

![KmpForm](/images/logo.png "KmpForm")

# TODO: Update with KMP gradle instructions
## Getting Started
To start using the library in your Android Compose project, follow these steps:
1. Add the JitPack repository to your settings.gradle file
```kotlin
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        // ...
        maven { url "https://jitpack.io" }
    }
}
```
> Note: In older Android projects, the repositories are defined in the root build.gradle file.

2. Add the dependency in your build.gradle file.
```kotlin
implementation 'com.steeplesoft:kmp-form:0.3.0-SNAPSHOT'
```

## Easy example
In a first example we create a simple form with two text fields. The form will look like this:

![KmpForm Simple](/screenshots/png/simple-form.png "Simple Form")

1. Create a your form class with your form field annotations (`@FormField`)
```kotlin
class MainForm(): Form() {
    override fun self(): Form {
        return this
    }

    override fun getFormFields() = listOf(name, lastName)

    @FormField
    val name = FieldState(
        state = mutableStateOf<String?>(null),
        validators = mutableListOf(NotEmptyValidator())
    )

    @FormField
    val lastName = FieldState(
        state = mutableStateOf<String?>(null)
    )
}
```
# TODO - Hilt-free example
2. Create a ViewModel for your form.
```kotlin
@HiltViewModel
class MainViewModel @Inject constructor(
    resourcesProvider: ResourcesProvider
): ViewModel() {
    var form = MainForm(resourcesProvider)

    fun validate() {
        form.validate(true)
        Log.d("MainViewModel", "Validate (form is valid: ${form.isValid})")
    }
}
```
3. Add the fields in your composable UI.
```kotlin
Column {
    TextField(
        label = "Name",
        form = viewModel.form,
        fieldState = viewModel.form.name,
    ).Field()

    TextField(
        label = "Last Name",
        form = viewModel.form,
        fieldState = viewModel.form.lastName
    ).Field()
}
```

## Extended Form example
We now try to make a more complex form with different validators, date fields, password fields and
searchable pickers. This is how the form will look like:

![KmpForm Extended](/screenshots/gif/KmpForm-extended.gif "Extended Form")

1. Create a form class with form fields. Define form fields by the `@FormField` annotation.
```kotlin
// in this example we have a separate data class `Country` for a country picker.
data class Country(
    val code: String,
    val name: String
): PickerValue() {
    override fun searchFilter(query: String): Boolean {
        return this.name.startsWith(query)
    }
}

class MainForm(resourcesProvider: ResourcesProvider): Form() {
    override fun self(): Form {
        return this
    }

    override fun getFormFields() = listOf(name, lastName, password, passwordConfirm, email,
        country, startDate, endDate, agreeWithTerms)

    @FormField
    val name = FieldState(
        state = mutableStateOf<String?>(null),
        validators = mutableListOf(
            NotEmptyValidator(),
            MinLengthValidator(
                minLength = 3,
                errorText = resourcesProvider.getString(R.string.error_min_length)
            )
        )
    )

    @FormField
    val lastName = FieldState(
        state = mutableStateOf<String?>(null)
    )

    @FormField
    val password = FieldState(
        state = mutableStateOf<String?>(null),
        validators = mutableListOf(
            NotEmptyValidator(),
            MinLengthValidator(
                minLength = 8,
                errorText = resourcesProvider.getString(R.string.error_min_length)
            )
        )
    )

    @FormField
    val passwordConfirm = FieldState(
        state = mutableStateOf<String?>(null),
        validators = mutableListOf(
            IsEqualValidator({ password.state.value })
        )
    )

    @FormField
    val email = FieldState(
        state = mutableStateOf<String?>(null),
        validators = mutableListOf(
            EmailValidator()
        )
    )

    @FormField
    val country = FieldState(
        state = mutableStateOf<Country?>(null),
        options = mutableListOf(
            Country(code = "CH", name = "Switzerland"),
            Country(code = "DE", name = "Germany"),
            Country(code = "FR", name = "France"),
            Country(code = "US", name = "United States"),
            Country(code = "ES", name = "Spain"),
            Country(code = "BR", name = "Brazil"),
            Country(code = "CN", name = "China"),
        ),
        optionItemFormatter = { "${it?.name}" },
        validators = mutableListOf(
            NotEmptyValidator()
        )
    )

    @FormField
    val startDate = FieldState(
        state = mutableStateOf<Date?>(null),
        validators = mutableListOf(
            NotEmptyValidator()
        )
    )

    @FormField
    val endDate = FieldState(
        state = mutableStateOf<Date?>(null),
        validators = mutableListOf(
            NotEmptyValidator(),
            DateValidator(
                minDateTime = {startDate.state.value?.time ?: 0},
                errorText = resourcesProvider.getString(R.string.error_date_after_start_date)
            )
        )
    )

    @FormField
    val agreeWithTerms = FieldState(
        state = mutableStateOf<Boolean?>(null),
        validators = mutableListOf(
            IsEqualValidator({ true })
        )
    )
}
```
2. Create a ViewModel for your form.
```kotlin
@HiltViewModel
class MainViewModel @Inject constructor(
    resourcesProvider: ResourcesProvider
): ViewModel() {
    var form = MainForm(resourcesProvider)

    fun validate() {
        form.validate(true)
        Log.d("MainViewModel", "Validate (form is valid: ${form.isValid})")
    }
}
```
3. Add the fields in your composable UI.
```kotlin
Column {
    TextField(
        modifier = Modifier.padding(bottom = 8.dp),
        label = "Name",
        form = viewModel.form,
        fieldState = viewModel.form.name,
    ).Field()

    TextField(
        modifier = Modifier.padding(bottom = 8.dp),
        label = "E-Mail",
        form = viewModel.form,
        fieldState = viewModel.form.email,
        keyboardType = KeyboardType.Email
    ).Field()

    PasswordField(
        modifier = Modifier.padding(bottom = 8.dp),
        label = "Password",
        form = viewModel.form,
        fieldState = viewModel.form.password
    ).Field()

    PasswordField(
        modifier = Modifier.padding(bottom = 8.dp),
        label = "Password Confirm",
        form = viewModel.form,
        fieldState = viewModel.form.passwordConfirm
    ).Field()

    TextField(
        modifier = Modifier.padding(bottom = 8.dp),
        label = "Last Name",
        form = viewModel.form,
        fieldState = viewModel.form.lastName,
        isEnabled = false,
    ).Field()

    PickerField(
        modifier = Modifier.padding(bottom = 8.dp),
        label = "Country",
        form = viewModel.form,
        fieldState = viewModel.form.country
    ).Field()

    DateField(
        modifier = Modifier.padding(bottom = 8.dp),
        label = "Start Date",
        form = viewModel.form,
        fieldState = viewModel.form.startDate,
        formatter = ::dateShort
    ).Field()

    DateField(
        modifier = Modifier.padding(bottom = 8.dp),
        label = "End Date",
        form = viewModel.form,
        fieldState = viewModel.form.endDate,
        formatter = ::dateLong
    ).Field()

    CheckboxField(
        modifier = Modifier.padding(bottom = 8.dp),
        fieldState = viewModel.form.agreeWithTerms,
        label = "I agree to Terms & Conditions",
        form = viewModel.form
    ).Field()
}
```

## Features
* A variety of form fields to choose from, including text input, pickers, checkbox, and more
* Built-in validators to ensure accurate user input
* Data binding for easy management of form data in your code

## License
This library is licensed under the MIT License.
