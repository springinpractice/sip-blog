---
layout: post
title: "How to pretty print your JSON with Spring and Jackson"
date: 2013-11-01 21:29
comments: true
categories: [ "Chapter 13 - Integration", "Quick Tips" ]
---
When implementing RESTful web services using Spring Web MVC, by default Spring doesn't pretty print the JSON output. Fortunately this is easy to address when using Java-based configuration. Here's how, using Jackson 2.

<!-- more -->

    @Configuration
    public class MyAppConfig extends WebMvcConfigurerAdapter {

        @Override
        public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
            StringHttpMessageConverter stringConverter = new StringHttpMessageConverter();
            stringConverter.setWriteAcceptCharset(false);
            converters.add(new ByteArrayHttpMessageConverter());
            converters.add(stringConverter);
            converters.add(new ResourceHttpMessageConverter());
            converters.add(new SourceHttpMessageConverter<Source>());
            converters.add(new AllEncompassingFormHttpMessageConverter());
            converters.add(jackson2Converter());
        }

        @Bean
        public MappingJackson2HttpMessageConverter jackson2Converter() {
            MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter();
            converter.setObjectMapper(objectMapper());
            return converter;
        }

        @Bean
        public ObjectMapper objectMapper() {
            Object objectMapper = new ObjectMapper();
            objectMapper.enable(SerializationFeature.INDENT_OUTPUT);
            return objectMapper;
        }
    }

Here we can see a key advantage of using Java-based configuration, which is that we can easily call the `ObjectMapper.enable()` method, allowing us to activate pretty printing.
