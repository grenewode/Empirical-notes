Thoughts on the UI:

There is no point to doing anything with the static map. It is better to do it at runtime, although something may be possible with static defaults?

Overview of the structure of things:

## Attempt 1 ##

This is potentially efficient, but may have problems later...

struct has_color_t {
  static color_t default_color;
  defaulted_value<color_t, &has_color_t::default_color> color;
};

## Attempt 2 ##

class rect_t {
  dvalue_t<color_t> background_color;
  dvalue_t<color_t> color;
  
  template<typename P>
  void render(const P& parent) const {
    // TODO: this *will* result in unnesessary copeies :-(
    auto color = this->color(parent.color);
    auto background_color = this->background_color(parent.color);
    gl...
  }
};



it would be good to have some sort of handle to shaders...
eg:
class renderer {
  template<class Color, class... Args>
  void render(const Color& color, Args&& ... args) {
    vao.push(std::forward<Args>(args)...);
    shader.uniform(color);
    vao.draw();
  }
}

The problem is that, while it generally makes sense for each element to handle rendering itself, in some cases it will be better to have them render as groups.

So, you need something like

template<class T>
class batchrenderer;

class datapoint_t {
  float x, y;
  dvalue_t<color_t> background_color;
  dvalue_t<color_t> color;

  
}

template<>
class batchrenderer<datapoint_t> {
  template<class P, class I>
  void render_all(const P& parent, I begin, I end) {
    std::vector<typename I::value_type::render_info_t> renderable;
    renderable.reserve(end - begin);

    std::transform(begin, end, std::back_inserter{renderable}, [parent](auto& item) {return item.render(parent); });
    // Do gl rendering things
    // Also, we should really be doing something with
  }
}

So, the real problem is twofold:
1. How to ensure that the transform is shared between all shaders
2. How to pass styling info into the shaders?

I think that the right way to deal with this is to create type safe wrappers around renderers.

So, what is a renderer? NO, that's the wrong word. I want a *rendering context* instead. That is much better, because it accounts for pixel data being bound to this thing. An even better word is drawingcontext.

NO! there are two distinct things here, I think. First, there is the Renderer. This just defines an interface for talking to a GPU shader object. Then this can be handed to the DrawingContext, which is used to do the stuff you want?

class MyRenderer {
  private:
  VertexArrayObject vao;
  ShaderProgram shader;
  public:
  MyRenderer(const GLcanvas& canvas) {
    vao
  }
}

So, you now have this:

class  {
  void push
}