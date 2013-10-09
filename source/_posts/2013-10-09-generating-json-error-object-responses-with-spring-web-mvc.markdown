---
layout: post
title: "Generating JSON error object responses with Spring Web MVC"
date: 2013-10-09 00:39
comments: true
categories: [ "Chapter 13 - Integration", "Quick Tips" ]
---
The other day I wrote a post called [Handling JSON error object responses with Spring's RestTemplate](http://springinpractice.com/2013/10/07/handling-json-error-object-responses-with-springs-resttemplate/). Judging by the Twitter activity, people found it useful, so this time around I'm going to write about the other side of the equation, which is *generating* the JSON error objects using Spring Web MVC. (For a quick background on what this means and why we'd want to do it, please see the post I just linked to.)

There are various ways to do this, but Spring 3.2 introduces a pretty elegant approach via the `@ControllerAdvice` annotation. The basic concept here is that we can define AOP-like "advice" around Spring Web MVC controllers. This advice captures exceptions and then maps them to JSON objects, which the advice sends in the response body. Of course we can also send the appropriate HTTP status code in the headers too.

<!-- more -->

(You can find out more about `@ControllerAdvice` and `@ExceptionHandler` in the post [Error Handling for REST with Spring 3](http://www.baeldung.com/2013/01/31/exception-handling-for-rest-with-spring-3-2/) by Eugen Paraschiv.)

Note that the error-triggering event doesn't really have to be an exception *per se*. For example, we might want bean validation errors or authorization errors&mdash;neither of which manifests itself as an exception&mdash;to map to JSON error objects. The key is to have these triggers generate exceptions that we can capture using the `@ControllerAdvice` component.

Let's look at an example involving bean validation. Here we have a controller. If the incoming resource is invalid, we want to generate a JSON error object. So first we do this:

    package myapp.web.controller;
    
    ... various imports ...
    
    @Controller
    @RequestMapping("/doodads")
    public class DoodadController {
        @Inject private DoodadService doodadService;
        
        @RequestMapping(
                value = "/{id}",
                method = RequestMethod.PUT,
                consumes = MediaType.APPLICATION_JSON_VALUE)
        public void updateDoodad(
                @PathVariable Long id,
                @RequestBody @Valid DoodadResource doodad,
                BindingResult bindingResult) {
            
            if (bindingResult.hasErrors()) {
                throw new InvalidRequestException("Invalid doodad", bindingResult);
            }
            
            doodadService.updateDoodad(doodad);
        }   
    }

Note that `InvalidRequestException` is just a custom exception I created that takes an `Errors` object as an argument. (`BindingResult` implements `Errors`.) Just for completeness, here's `InvalidRequestException`:

    package myapp.exception;
    
    import org.springframework.validation.Errors;
    
    @SuppressWarnings("serial")
    public class InvalidRequestException extends RuntimeException {
        private Errors errors;
        
        public InvalidRequestException(String message, Errors errors) {
            super(message);
            this.errors = errors;
        }
        
        public Errors getErrors() { return errors; }
    }

So far so good. But now we need that `@ControllerAdvice` to capture the `InvalidRequestException` and generate the JSON error object:

    package myapp.web.controller;
    
    import java.util.ArrayList;
    import java.util.List;
    
    import org.springframework.http.HttpHeaders;
    import org.springframework.http.HttpStatus;
    import org.springframework.http.MediaType;
    import org.springframework.http.ResponseEntity;
    import org.springframework.validation.FieldError;
    import org.springframework.web.bind.annotation.ControllerAdvice;
    import org.springframework.web.bind.annotation.ExceptionHandler;
    import org.springframework.web.context.request.WebRequest;
    import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;
    
    import myapp.binding.ErrorResource;
    import myapp.binding.FieldErrorResource;
    import myapp.exception.InvalidRequestException;
    
    @ControllerAdvice
    public class MyExceptionHandler extends ResponseEntityExceptionHandler {
        
        @ExceptionHandler({ InvalidRequestException.class })
        protected ResponseEntity<Object> handleInvalidRequest(RuntimeException e, WebRequest request) {
            InvalidRequestException ire = (InvalidRequestException) e;
            List<FieldErrorResource> fieldErrorResources = new ArrayList<>();
            
            List<FieldError> fieldErrors = ire.getErrors().getFieldErrors();
            for (FieldError fieldError : fieldErrors) {
                FieldErrorResource fieldErrorResource = new FieldErrorResource();
                fieldErrorResource.setResource(fieldError.getObjectName());
                fieldErrorResource.setField(fieldError.getField());
                fieldErrorResource.setCode(fieldError.getCode());
                fieldErrorResource.setMessage(fieldError.getDefaultMessage());
                fieldErrorResources.add(fieldErrorResource);
            }
            
            ErrorResource error = new ErrorResource("InvalidRequest", ire.getMessage());
            error.setFieldErrors(fieldErrorResources);
    
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_JSON);
            
            return handleExceptionInternal(e, error, headers, HttpStatus.UNPROCESSABLE_ENTITY, request);
        }
        
        ... other handlers for other exceptions ...
    }

The important pieces here are `@ControllerAdvice` (which derives from `@Controller`, so we can component scan it), `ResponseEntityExceptionHandler` (provides the `handleExceptionInternal()` method), and `@ExceptionHandler` annotation. `@ExceptionHandler` accepts an array of match exceptions, and then its implementation builds the JSON error object, which here involves custom `ErrorResource` and `FieldErrorResource` beans that can be whatever we want to display to the client. Finally we pass response-related information to `handleExceptionInternal()`, where the error object ends up as the response body.

Again in the interest of completeness, here are the error objects I'm using. These are just examples of what's possible; choose error representations that fit your needs. First, the top-level error object:

    package myapp.binding;
    
    import java.util.List;
    import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
    
    @JsonIgnoreProperties(ignoreUnknown = true)
    public class ErrorResource {
        private String code;
        private String message;
        private List<FieldErrorResource> fieldErrors;
        
        public ErrorResource() { }
        
        public ErrorResource(String code, String message) {
            this.code = code;
            this.message = message;
        }
        
        public String getCode() { return code; }
        
        public void setCode(String code) { this.code = code; }
        
        public String getMessage() { return message; }
        
        public void setMessage(String message) { this.message = message; }
        
        public List<FieldErrorResource> getFieldErrors() { return fieldErrors; }
        
        public void setFieldErrors(List<FieldErrorResource> fieldErrors) {
            this.fieldErrors = fieldErrors;
        }
    }

And finally here's a class for field errors:

    package myapp.binding;
    
    import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
    
    @JsonIgnoreProperties(ignoreUnknown = true)
    public class FieldErrorResource {
        private String resource;
        private String field;
        private String code;
        private String message;
        
        public String getResource() { return resource; }
        
        public void setResource(String resource) { this.resource = resource; }
        
        public String getField() { return field; }
        
        public void setField(String field) { this.field = field; }
        
        public String getCode() { return code; }
        
        public void setCode(String code) { this.code = code; }
        
        public String getMessage() { return message; }
        
        public void setMessage(String message) { this.message = message; }
    }

Have fun!
