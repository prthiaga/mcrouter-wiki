<h2 id="json-with-macros-jsonm">JSON with macros (JSONM) <a href="#json-with-macros-jsonm" class="headerLink">#</a></h2>

<p>JSONM is a JSON extension that allows using macro definitions inside JSON files.</p>

<h3 id="why">Why? <a href="#why" class="headerLink">#</a></h3>

<p>The main goals of this format are to make configuration files more readable and to get rid
of scripts that generate huge configuration files.</p>

<h3 id="what">What? <a href="#what" class="headerLink">#</a></h3>

<p>JSONM is a superset of JSON. Any JSON object may be treated as JSON with
macros. JSON with macros is also a valid JSON object &#x2013; the difference lies in enhancing some JSON properties to allow
reuse of similar parts of large JSON objects.</p>

<h3 id="how">How? <a href="#how" class="headerLink">#</a></h3>

<p>Using a simple preprocessor, we generate standard JSON objects from
user-friendly JSONM by processing all macros and substituting all constants.</p>

<h2 id="syntax-definitions">Syntax &amp; Definitions <a href="#syntax-definitions" class="headerLink">#</a></h2>

<h4 id="comments">Comments <a href="#comments" class="headerLink">#</a></h4>

<p>JSONM allows C-style comments, which are removed by
the preprocessor. Example:</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="c">// some comment here</span>
  <span class="s2">&quot;key&quot;</span><span class="o">:</span> <span class="cm">/* and one more comment here */</span> <span class="s2">&quot;value/**/&quot;</span>
<span class="o">&#125;</span></pre></div>

<p>After preprocessing:</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;key&quot;</span><span class="o">:</span> <span class="s2">&quot;value/**/&quot;</span>
<span class="o">&#125;</span></pre></div>

<h4 id="escaping">Escaping <a href="#escaping" class="headerLink">#</a></h4>

<p>All fields in JSONM are preprocessed and characters <tt>&#064;</tt>, <tt>%</tt>, <tt>(</tt>, <tt>)</tt> and <tt>,</tt> have special meaning. You need to add two backslashes (<tt>\\</tt>) before any such character to avoid it being interpreted as a preprocessor instruction. To escape a backslash, write <tt>\\\\</tt>. Two backslashes are required because JSON uses a single backslash (<tt>\</tt>) as an escape character.</p>

<h4 id="value">value <a href="#value" class="headerLink">#</a></h4>

<p>Any JSON value (string, object, number, etc.)</p>

<h4 id="paramsubstitution">paramSubstitution <a href="#paramsubstitution" class="headerLink">#</a></h4>

<p>A string of form <tt>%paramName%</tt> that will be replaced by the value of that parameter.</p>

<p><strong>Examples:</strong> <tt>&quot;%paramName%&quot;</tt>, <tt>&quot;a%paramName%b&quot;</tt>. <tt>%paramName%</tt> will be replaced with a value of the corresponding parameter.</p>

<p>If the whole string is a parameter substitution (i.e., &quot;%paramName%&quot;) parameter values may be any valueWithParams. Otherwise, it should be a string.</p>

<h4 id="valuewithparams">valueWithParams <a href="#valuewithparams" class="headerLink">#</a></h4>

<p>A <tt>value</tt> that may contain <tt>paramSubstitution</tt>s in it.</p>

<h4 id="valuewithmacros">valueWithMacros <a href="#valuewithmacros" class="headerLink">#</a></h4>

<p>A <tt>value</tt> that may contain <tt>macroCall</tt>s and <tt>paramSubstitution</tt>s in it.</p>

<h4 id="macro">macro <a href="#macro" class="headerLink">#</a></h4>

<p>A reusable piece of JSON. One can think about it as a function that takes an arbitrary list of values and returns <tt>valueWithMacros</tt>.</p>

<h4 id="macrocall">macroCall <a href="#macrocall" class="headerLink">#</a></h4>

<p>A call of macro that would replace that call with its result. Most macros have both inline and expanded versions, but several macros have only expanded version.</p>

<p><strong>Inline form:</strong> <tt>&quot;&#064;macroName(param1,param2,&#x2026;)&quot;</tt>. Note: during preprocessing of inline form spaces around special characters are stripped; all arguments that are not macro calls are treated as strings. I.e. a call <tt>&quot; &#064;if(&#064;bool(true), foo, bar) &quot;</tt> is equivalent to <tt>&quot;&#064;if(&#064;bool(true),foo,bar)&quot;</tt>, also note explicit cast of string &quot;true&quot; to a boolean, without that cast the preprocessing will fail due to wrong argument type.</p>

<p><strong>Expanded form:</strong> an object with macro parameters passed as properties of that object plus next special properties</p>

<ul>
<li><tt>type</tt> string (required): macro name to call</li>
<li><tt>vars</tt> valueWithMacros that get preprocessed to an object (optional): allows to define additional context variables that would be available during preprocessing of parameters for the macro</li>
</ul>

<p><strong>Example:</strong> calling a call to a built-in macro <tt>if</tt> in extended form</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;if&quot;</span><span class="o">,</span>
  <span class="s2">&quot;vars&quot;</span><span class="o">:</span> <span class="o">&#123;</span>
    <span class="s2">&quot;A&quot;</span><span class="o">:</span> <span class="mi">5</span>
  <span class="o">&#125;,</span>
  <span class="s2">&quot;condition&quot;</span><span class="o">:</span> <span class="kc">true</span><span class="o">,</span>
  <span class="s2">&quot;is_true&quot;</span><span class="o">:</span> <span class="s2">&quot;&#064;add(%A%, 1)&quot;</span><span class="o">,</span>
  <span class="s2">&quot;is_false&quot;</span><span class="o">:</span> <span class="s2">&quot;&#064;add(%A%, 2)&quot;</span><span class="o">,</span>
<span class="o">&#125;</span></pre></div>

<p>here we have three required parameters (<tt>condition</tt>, <tt>is_true</tt>, <tt>is_false</tt>) for the macro <tt>if</tt>, we also define a local variable <tt>A</tt>. We can write similar code in an inline form <tt>&#064;if(&#064;bool(true),&#064;add(5,1),&#064;add(5,2))</tt>, though we don&#039;t have a way to define local vars.</p>

<h4 id="macrodefinition">macroDefinition <a href="#macrodefinition" class="headerLink">#</a></h4>

<p>Defines a reusable <tt>macro</tt>. Should be an object that is <tt>valueWithMacros</tt> and has next properties in it:</p>

<ul>
<li><tt>type</tt> string (required): should be set to <tt>&quot;macroDef&quot;</tt></li>
<li><tt>params</tt> list (optional): defines a list of parameters that this macro accepts, each param is either a string name of the parameter or an object with the following properties:<ul>
<li><tt>name</tt> string (required): name of the parameter</li>
<li><tt>optional</tt> bool (optional): marks the parameter as optional (i.e. it can be omitted when calling the macro)</li>
<li><tt>default</tt> valueWithMacros (optional): automatically implies <tt>optional = true</tt>, defines a default value that should be used for the parameter if it&#039;s not specified in <tt>macroCall</tt></li>
</ul></li>
<li><tt>result</tt> valueWithMacros (required): defines the return value of the macro, contents of the <tt>result</tt> can reference parameters specified in <tt>params</tt> list.</li>
</ul>

<p>Note: if the parameter is defined as optional, all following parameters should be defined as <tt>optional</tt> too.</p>

<h4 id="constdefinition">constDefinition <a href="#constdefinition" class="headerLink">#</a></h4>

<p>An object that is <tt>valueWithMacros</tt>. Defines a reusable constant that can be used in <tt>paramSubstitution</tt> in the same way as params, but on global scope. Should contain next properties:</p>

<ul>
<li><tt>type</tt> string (required): should be set to <tt>&quot;constDef&quot;</tt></li>
<li><tt>result</tt> valueWithMacros (required): the value of the constant</li>
</ul>

<p><strong>Example:</strong></p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;constDef&quot;</span><span class="o">,</span>
  <span class="s2">&quot;result&quot;</span><span class="o">:</span> <span class="o">&#123;</span>
    <span class="s2">&quot;foo&quot;</span><span class="o">:</span> <span class="s2">&quot;bar&quot;</span>
  <span class="o">&#125;</span>
<span class="o">&#125;</span></pre></div>

<h4 id="macrodefinitions">macroDefinitions <a href="#macrodefinitions" class="headerLink">#</a></h4>

<p>Set of <tt>macroDefinition</tt>s and <tt>constDefinitions</tt>. Can be either an object of type <tt>valueWithMacros</tt> or a recursive list of such objects. During preprocessing the list is flattened and objects are merged to form a final dictionary of definitions with keys being names of the defined <tt>macro</tt> or constants.</p>

<p>Example:</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">[</span>
  <span class="o">[</span>
    <span class="o">&#123;</span>
      <span class="s2">&quot;isOdd&quot;</span><span class="o">:</span> <span class="o">&#123;</span>
        <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;macroDef&quot;</span><span class="o">,</span>
        <span class="s2">&quot;params&quot;</span><span class="o">:</span> <span class="o">[</span> <span class="s2">&quot;value&quot;</span> <span class="o">],</span>
        <span class="s2">&quot;result&quot;</span><span class="o">:</span> <span class="s2">&quot;&#064;equals(&#064;mod(%value%,2),1)&quot;</span>
      <span class="o">&#125;</span>
    <span class="o">&#125;</span>
  <span class="o">],</span>
  <span class="o">[</span>
    <span class="o">&#123;</span>
      <span class="s2">&quot;dayInSeconds&quot;</span><span class="o">:</span> <span class="o">&#123;</span>
        <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;constDef&quot;</span><span class="o">,</span>
        <span class="s2">&quot;result&quot;</span><span class="o">:</span> <span class="s2">&quot;&#064;mul(3600,24)&quot;</span>
      <span class="o">&#125;</span>
    <span class="o">&#125;</span>
  <span class="o">]</span>
<span class="o">]</span></pre></div>

<h4 id="jsonm">JSONM <a href="#jsonm" class="headerLink">#</a></h4>

<p>JSONM is a JSON object with a special optional key <tt>macros</tt>:</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;macros&quot;</span><span class="o">:</span> <span class="no">macroDefinitions</span><span class="o">,</span>
  <span class="s2">&quot;property1&quot;</span><span class="o">:</span> <span class="no">valueWithMacros</span><span class="o">,</span>
  <span class="s2">&quot;property2&quot;</span><span class="o">:</span> <span class="no">valueWithMacros</span>
<span class="o">&#125;</span></pre></div>

<p>During the preprocessing all <tt>macroDefinitions</tt> and <tt>constDefinitions</tt> from <tt>&quot;macros&quot;</tt> are imported and afterward all properties are expanded with them.</p>

<h2 id="macro-library">Macro Library <a href="#macro-library" class="headerLink">#</a></h2>

<p>JSONM comes with a set of predefined functional macros that extend JSONM to a functional language.</p>

<h3 id="index">Index <a href="#index" class="headerLink">#</a></h3>

<ul>
<li><a href="#import"><tt>&#064;import</tt></a></li>
<li><a href="#int"><tt>&#064;int</tt></a></li>
<li><a href="#double"><tt>&#064;double</tt></a></li>
<li><a href="#bool"><tt>&#064;bool</tt></a></li>
<li><a href="#str"><tt>&#064;str</tt></a></li>
<li><a href="#not"><tt>&#064;not</tt></a></li>
<li><a href="#and"><tt>&#064;and</tt></a></li>
<li><a href="#or"><tt>&#064;or</tt></a></li>
<li><a href="#less"><tt>&#064;less</tt></a></li>
<li><a href="#equals"><tt>&#064;equals</tt></a></li>
<li><a href="#if"><tt>&#064;if</tt></a></li>
<li><a href="#isBool"><tt>&#064;isBool</tt></a></li>
<li><a href="#isInt"><tt>&#064;isInt</tt></a></li>
<li><a href="#isDouble"><tt>&#064;isDouble</tt></a></li>
<li><a href="#isString"><tt>&#064;isString</tt></a></li>
<li><a href="#isArray"><tt>&#064;isArray</tt></a></li>
<li><a href="#isObject"><tt>&#064;isObject</tt></a></li>
<li><a href="#add"><tt>&#064;add</tt></a></li>
<li><a href="#sub"><tt>&#064;sub</tt></a></li>
<li><a href="#mul"><tt>&#064;mul</tt></a></li>
<li><a href="#div"><tt>&#064;div</tt></a></li>
<li><a href="#mod"><tt>&#064;mod</tt></a></li>
<li><a href="#empty"><tt>&#064;empty</tt></a></li>
<li><a href="#size"><tt>&#064;size</tt></a></li>
<li><a href="#contains"><tt>&#064;contains</tt></a></li>
<li><a href="#keys"><tt>&#064;keys</tt></a></li>
<li><a href="#values"><tt>&#064;values</tt></a></li>
<li><a href="#select"><tt>&#064;select</tt></a></li>
<li><a href="#set"><tt>&#064;set</tt></a></li>
<li><a href="#shuffle"><tt>&#064;shuffle</tt></a></li>
<li><a href="#merge"><tt>&#064;merge</tt></a></li>
<li><a href="#slice"><tt>&#064;slice</tt></a></li>
<li><a href="#sort"><tt>&#064;sort</tt></a></li>
<li><a href="#split"><tt>&#064;split</tt></a></li>
<li><a href="#transform"><tt>&#064;transform</tt> (no-inline)</a></li>
<li><a href="#foreach"><tt>&#064;foreach</tt> (no-inline)</a></li>
<li><a href="#process"><tt>&#064;process</tt> (no-inline)</a></li>
<li><a href="#define"><tt>&#064;define</tt></a></li>
<li><a href="#range"><tt>&#064;range</tt></a></li>
<li><a href="#defined"><tt>&#064;defined</tt></a></li>
<li><a href="#fail"><tt>&#064;fail</tt></a></li>
<li><a href="#isLocalIp"><tt>&#064;isLocalIp</tt></a></li>
<li><a href="#hash"><tt>&#064;hash</tt></a></li>
<li><a href="#weightedHash"><tt>&#064;weightedHash</tt></a></li>
</ul>

<span id="import"></span>

<h3 id="import-other-json-file"><tt>&#064;import</tt> other JSON file <a href="#import-other-json-file" class="headerLink">#</a></h3>

<p>One of the key features of JSONM is that it allows you to make your configs more modular or depend on external configuration sources. <tt>&#064;import</tt> macro reads some external resource and gets replaced with the contents of that resource. JSONM preprocessor requires a caller to pass in an ImportResolver object that knows how to resolve resource name to its contents. The convention for the resource name is <tt>&quot;resource_type:name&quot;</tt> (e.g. <tt>&quot;file:myFile.json&quot;</tt>), this is supported via mcrouter ConfigApi that would see the dependency and monitor changes to it.</p>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>path</tt> valueWithMacros that evaluates to string (required): resource identifier (file path)</li>
<li><tt>default</tt> valueWithMacros (optional): default value to use on import failure, if not set import failure would result in preprocessing failure</li>
</ul>

<p><strong>Return:</strong> content of the external resource.</p>

<p><strong>Example:</strong> assume we have two files as below</p>

<blockquote><p>citiesByCountry.json</p></blockquote>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;ukraine&quot;</span><span class="o">:</span> <span class="o">[</span> <span class="s2">&quot;Kyiv&quot;</span><span class="o">,</span> <span class="s2">&quot;Lviv&quot;</span> <span class="o">],</span>
  <span class="s2">&quot;usa&quot;</span><span class="o">:</span> <span class="o">[</span> <span class="s2">&quot;Menlo Park&quot;</span> <span class="o">]</span>
<span class="o">&#125;</span></pre></div>

<blockquote><p>allCities.json</p></blockquote>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;cities&quot;</span><span class="o">:</span> <span class="s2">&quot;&#064;merge(&#064;values(&#064;import(file:citiesByCountry.json)))&quot;</span>
<span class="o">&#125;</span></pre></div>

<p>after preprocessing, <tt>allCities.json</tt> would result in</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;cities&quot;</span><span class="o">:</span> <span class="o">[</span> <span class="s2">&quot;Kyiv&quot;</span><span class="o">,</span> <span class="s2">&quot;Lviv&quot;</span><span class="o">,</span> <span class="s2">&quot;Menlo Park&quot;</span> <span class="o">]</span>
<span class="o">&#125;</span></pre></div>

<span id="int"></span><span id="bool"></span><span id="str"></span>

<h3 id="type-conversions-int-boo">Type conversions <tt>&#064;int, &#064;double, &#064;bool, &#064;str</tt> <a href="#type-conversions-int-boo" class="headerLink">#</a></h3>

<p>JSONM allows you to do type conversion between integer, boolean and string types. These four macro convert input <tt>value</tt> parameter to the integer, double, boolean, string respectively. Conversion happens according to the <tt>folly::to&lt;T&gt;(arg)</tt> logic.</p>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>value</tt> valueWithMacros (required): a value to convert.</li>
</ul>

<p><strong>Examples:</strong> <tt>&#064;int(123)</tt>, <tt>&#064;double(5.5)</tt>, <tt>&#064;bool(1)</tt>, <tt>&#064;bool(true)</tt>, <tt>&#064;bool(false)</tt>, <tt>&#064;str(12345)</tt>, <tt>&#064;str(true)</tt>.</p>

<span id="not"></span>

<h3 id="boolean-unary-not">Boolean unary <tt>&#064;not</tt> <a href="#boolean-unary-not" class="headerLink">#</a></h3>

<p>Returns a negation of argument. I.e. converts <tt>true</tt> to <tt>false</tt> and <tt>false</tt> to <tt>true</tt>.</p>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>A</tt> valueWithMacros that evaluates to boolean (required)</li>
</ul>

<p><strong>Return:</strong> <tt>not A</tt>.</p>

<p><strong>Examples:</strong> <tt>&#064;not(true)</tt> would result in <tt>false</tt>; <tt>&#064;not(&#064;equals(1, 2))</tt> would result in <tt>true</tt>.</p>

<span id="and"></span><span id="or"></span>

<h3 id="boolean-binary-and-or">Boolean binary <tt>&#064;and, &#064;or</tt> <a href="#boolean-binary-and-or" class="headerLink">#</a></h3>

<p>Macros that represents logical conjunction and disjunction.</p>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>A</tt> valueWithMacros that evaluates to boolean (required)</li>
<li><tt>B</tt> valueWithMacros that evaluates to boolean (required)</li>
</ul>

<p><strong>Return:</strong> <tt>A and B</tt> and <tt>A or B</tt> respectively.</p>

<p><strong>Examples:</strong> <tt>&#064;and(&#064;not(true), false)</tt> would result in <tt>false</tt>; <tt>&#064;or(false, true)</tt> would result in <tt>true</tt>.</p>

<span id="less"></span><span id="equals"></span>

<h3 id="comparison-less-equals">Comparison <tt>&#064;less, &#064;equals</tt> <a href="#comparison-less-equals" class="headerLink">#</a></h3>

<p>These two macro allow you to compare values and return boolean.</p>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>A</tt> valueWithMacros (required): first argument</li>
<li><tt>B</tt> valueWithMacros (required): second argument</li>
</ul>

<p><strong>Return:</strong></p>

<ul>
<li>boolean, result of &#039;A &lt; B&#039; and &#039;A == B&#039; respectively.</li>
</ul>

<p><strong>Examples:</strong> <tt>&#064;less(5, 3)</tt> will return false, <tt>&#064;equals(true,true)</tt> will return true.</p>

<p><strong>Note:</strong> all other comparisons could be expressed by applying boolean operators. I.e. greater can be expressed as <tt>&#064;not(&#064;or(&#064;less(A,B),&#064;equals(A,B)))</tt>.</p>

<span id="if"></span>

<h3 id="conditional-macro-if">Conditional macro <tt>&#064;if</tt> <a href="#conditional-macro-if" class="headerLink">#</a></h3>

<p>Allows doing preprocessing based on some condition.</p>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>condition</tt> valueWithMacros that evaluates to boolean (required)</li>
<li><tt>is_true</tt> valueWithMacros (required): value to preprocess and return if <tt>condition</tt> evaluates to <tt>true</tt></li>
<li><tt>is_false</tt> valueWithMacros (required): value to preprocess and return if <tt>condition</tt> evaluates to <tt>false</tt></li>
</ul>

<p><strong>Return:</strong> <tt>is_true</tt> if <tt>condition</tt> evaluates to true or <tt>is_false</tt> otherwise.</p>

<p><strong>Examples:</strong> <tt>&#064;if(&#064;equals(1,1),same,different)</tt> would result in <tt>same</tt>,</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;if&quot;</span><span class="o">,</span>
  <span class="s2">&quot;condition&quot;</span><span class="o">:</span> <span class="s2">&quot;&#064;equals(1,0)&quot;</span><span class="o">,</span>
  <span class="s2">&quot;is_true&quot;</span><span class="o">:</span> <span class="o">[</span> <span class="s2">&quot;foo&quot;</span> <span class="o">],</span>
  <span class="s2">&quot;is_false&quot;</span><span class="o">:</span> <span class="o">[</span> <span class="s2">&quot;bar&quot;</span> <span class="o">]</span>
<span class="o">&#125;</span></pre></div>

<p>would result in <tt>[ &quot;bar&quot; ]</tt>.</p>

<span id="isBool"></span><span id="isInt"></span><span id="isDouble"></span><span id="isString"></span><span id="isArray"></span><span id="isObject"></span>

<h3 id="testing-type-of-a-value">Testing type of a value <tt>&#064;isBool, &#064;isInt, &#064;isDouble, &#064;isString, &#064;isArray, &#064;isObject</tt> <a href="#testing-type-of-a-value" class="headerLink">#</a></h3>

<p>These six macros allow you to check if the parameter is of a certain type (boolean, integer, string, array or an object respectively).</p>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>A</tt> valueWithMacros (required).</li>
</ul>

<p><strong>Return:</strong> boolean <tt>true</tt> if the parameter <tt>A</tt> is of the tested type, <tt>false</tt> otherwise.</p>

<p><strong>Examples:</strong> <tt>&#064;isInt(&#064;int(0))</tt> would result in <tt>true</tt>, <tt>&#064;isInt(false)</tt> would result in <tt>false</tt>.</p>

<span id="add"></span><span id="sub"></span><span id="mul"></span><span id="div"></span><span id="mod"></span>

<h3 id="arithmetic-binary-macros">Arithmetic binary macros <tt>&#064;add, &#064;sub, &#064;mul, &#064;div, &#064;mod</tt> <a href="#arithmetic-binary-macros" class="headerLink">#</a></h3>

<p>Perform arithmetic operation (summation, subtraction, multiplication, division or modulo respectively) on input parameters.</p>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>A</tt> valueWithMacros that evaluates to integer (required).</li>
<li><tt>B</tt> valueWithMacros that evaluates to integer (required).</li>
</ul>

<p><strong>Return:</strong> <tt>A+B</tt>, <tt>A-B</tt>, <tt>A*B</tt>, <tt>A/B</tt>, <tt>A%B</tt> respectively.</p>

<p><strong>Example:</strong> <tt>&#064;add(&#064;mod(5, 2), 1)</tt> would result in <tt>2</tt>.</p>

<h3 id="macro-for-working-with-c">Macro for working with compound types (strings, arrays, objects) <a href="#macro-for-working-with-c" class="headerLink">#</a></h3>

<span id="empty"></span>

<h3 id="check-if-the-value-is-em">Check if the value is <tt>&#064;empty</tt> <a href="#check-if-the-value-is-em" class="headerLink">#</a></h3>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>dictionary</tt> valueWithMacros that evaluates to string, array or object (required).</li>
</ul>

<p><strong>Return:</strong> <tt>true</tt> if the value is empty (containing 0 elements), <tt>false</tt> otherwise.</p>

<p><strong>Example:</strong> <tt>&#064;empty(&#x201c;&#x201d;)</tt> would evaluate to <tt>true</tt>.</p>

<span id="size"></span>

<h3 id="obtain-size-of-the-param">Obtain <tt>&#064;size</tt> of the parameter <a href="#obtain-size-of-the-param" class="headerLink">#</a></h3>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>dictionary</tt> valueWithMacros that evaluates to string, array or object (required).</li>
</ul>

<p><strong>Return:</strong> length of a string or a number of elements in array or object.</p>

<p><strong>Examples:</strong> <tt>&#064;size(abc)</tt> would return <tt>3</tt>.</p>

<span id="contains"></span>

<h3 id="check-if-the-input-param">Check if the input parameter <tt>&#064;contains</tt> given substring, value or key <a href="#check-if-the-input-param" class="headerLink">#</a></h3>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>dictionary</tt> valueWithMacros that evaluates to string, array or object (required)</li>
<li><tt>key</tt> valueWithMacros that evaluates to string if <tt>dictionary</tt> is a string or an object; any type in case of array (required).</li>
</ul>

<p><strong>Return:</strong> <tt>true</tt> if</p>

<ul>
<li><tt>dictionary</tt> is a <tt>string</tt> and it contains <tt>key</tt> as a substring</li>
<li><tt>dictionary</tt> is an <tt>array</tt> and it contains item that is equal to the <tt>key</tt></li>
<li><tt>dictionary</tt> is an <tt>object</tt> and it contains property with a name <tt>key</tt>,</li>
</ul>

<p><tt>false</tt> otherwise.</p>

<span id="keys"></span>

<h3 id="obtain-keys-of-propertie">Obtain <tt>&#064;keys</tt> of properties in an object <a href="#obtain-keys-of-propertie" class="headerLink">#</a></h3>

<p>Obtain keys of an object and return them as an array.</p>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>dictionary</tt> valueWithMacros that evaluates to an object (required).</li>
</ul>

<p><strong>Return:</strong> array of keys without any particular ordering (can be different for the equal input).</p>

<p><strong>Example:</strong></p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="no">&#x201c;type&#x201d;</span><span class="o">:</span> <span class="no">&#x201c;keys&#x201d;</span><span class="o">,</span>
  <span class="no">&#x201c;dictionary&#x201d;</span><span class="o">:</span> <span class="o">&#123;</span>
    <span class="no">&#x201c;foo&#x201d;</span><span class="o">:</span> <span class="no">&#x201c;bar&#x201d;</span>
  <span class="o">&#125;</span>
<span class="o">&#125;</span></pre></div>

<p>would return <tt>[ &#x201c;foo&#x201d; ]</tt>.</p>

<span id="values"></span>

<h3 id="obtain-values-of-propert">Obtain <tt>&#064;values</tt> of properties in an object <a href="#obtain-values-of-propert" class="headerLink">#</a></h3>

<p>For a given object returns an array containing values of all properties inside of that object.</p>

<p><strong>Parameters:</strong>
* <tt>dictionary</tt> valueWithMacros that evaluates to an object (required).</p>

<p><strong>Return:</strong> array of values without any particular ordering (can be different for the equal input).</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="no">&#x201c;type&#x201d;</span><span class="o">:</span> <span class="no">&#x201c;values&#x201d;</span><span class="o">,</span>
  <span class="no">&#x201c;dictionary&#x201d;</span><span class="o">:</span> <span class="o">&#123;</span>
    <span class="no">&#x201c;foo&#x201d;</span><span class="o">:</span> <span class="no">&#x201c;bar&#x201d;</span><span class="o">,</span>
    <span class="no">&#x201c;baz&#x201d;</span><span class="o">:</span> <span class="o">&#123;</span>
      <span class="no">&#x201c;abc&#x201d;</span><span class="o">:</span> <span class="mi">1</span>
    <span class="o">&#125;</span>
  <span class="o">&#125;</span>
<span class="o">&#125;</span></pre></div>

<p>would return</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">[</span>
  <span class="no">&#x201c;bar&#x201d;</span><span class="o">,</span>
  <span class="o">&#123;</span>
    <span class="no">&#x201c;abc&#x201d;</span><span class="o">:</span> <span class="mi">1</span>
  <span class="o">&#125;</span>
<span class="o">]</span></pre></div>

<span id="select"></span>

<h3 id="select-an-item-from-the"><tt>&#064;select</tt> an item from the input <a href="#select-an-item-from-the" class="headerLink">#</a></h3>

<p>Performs a lookup of item in an array (based on index) or in an object (based on key).</p>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>dictionary</tt> valueWithMacros that evaluates to an object or an array (required)</li>
<li><tt>key</tt> valueWithMacros that evaluates to a string for an object <tt>dictionary</tt> and integer for an array <tt>dictionary</tt> (required)</li>
<li><tt>default</tt> valueWithMacros (optional): a value that would be returned if the <tt>key</tt> is not found in <tt>dictionary</tt>.</li>
</ul>

<p><strong>Return:</strong> if the <tt>key</tt> is found in <tt>dictionary</tt> returns the value, otherwise if <tt>default</tt> is set returns <tt>default</tt>, else throws an error.</p>

<p><strong>Examples:</strong></p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="no">&#x201c;type&#x201d;</span><span class="o">:</span> <span class="no">&#x201c;select&#x201d;</span><span class="o">,</span>
  <span class="no">&#x201c;dictionary&#x201d;</span><span class="o">:</span> <span class="o">[</span> <span class="mi">1</span><span class="o">,</span> <span class="mi">2</span><span class="o">,</span> <span class="mi">3</span> <span class="o">],</span>
  <span class="no">&#x201c;key&#x201d;</span><span class="o">:</span> <span class="mi">0</span>
<span class="o">&#125;</span></pre></div>

<p>would result in <tt>1</tt>,</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="no">&#x201c;type&#x201d;</span><span class="o">:</span> <span class="no">&#x201c;select&#x201d;</span><span class="o">,</span>
  <span class="no">&#x201c;dictionary&#x201d;</span><span class="o">:</span> <span class="o">[</span> <span class="mi">1</span><span class="o">,</span> <span class="mi">2</span><span class="o">,</span> <span class="mi">3</span> <span class="o">],</span>
  <span class="no">&#x201c;key&#x201d;</span><span class="o">:</span> <span class="mi">3</span>
<span class="o">&#125;</span></pre></div>

<p>would produce an error,</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="no">&#x201c;type&#x201d;</span><span class="o">:</span> <span class="no">&#x201c;select&#x201d;</span><span class="o">,</span>
  <span class="no">&#x201c;dictionary&#x201d;</span><span class="o">:</span> <span class="o">&#123;</span>
    <span class="no">&#x201c;foo&#x201d;</span><span class="o">:</span> <span class="no">&#x201c;bar&#x201d;</span>
  <span class="o">&#125;,</span>
  <span class="no">&#x201c;key&#x201d;</span><span class="o">:</span> <span class="mi">&#x201c;foo&#x201d;</span>
<span class="o">&#125;</span></pre></div>

<p>would result in <tt>&#x201d;bar&#x201d;</tt>.</p>

<span id="set"></span>

<h3 id="create-a-copy-of-paramet">Create a copy of parameter with an item/property <tt>&#064;set</tt> in it <a href="#create-a-copy-of-paramet" class="headerLink">#</a></h3>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>dictionary</tt> valueWithMacros that evaluates to an object or an array (required)</li>
<li><tt>key</tt> valueWithMacros that evaluates to a string if the <tt>dictionary</tt> is object; in case an array <tt>dictionary</tt>, must be integer an be within the size ranges (i.e. 0 &lt;= <tt>key</tt> &lt; <tt>&#064;size(%dictionary%)</tt>); (required)</li>
<li><tt>value</tt> valueWithMacros (required)</li>
</ul>

<p><strong>Return:</strong> for <tt>dictionary</tt> that evaluates to an object, returns a copy of that dictionary with property <tt>key</tt> set to <tt>value</tt>; for a <tt>dictionary</tt> that evaluates to an array returns array with item at index <tt>key</tt> set to `value.</p>

<p><strong>Example:</strong></p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="no">&#x201c;type&#x201d;</span><span class="o">:</span> <span class="no">&#x201c;set&#x201d;</span><span class="o">,</span>
  <span class="no">&#x201c;dictionary&#x201d;</span><span class="o">:</span> <span class="o">&#123;</span>
    <span class="no">&#x201c;bar&#x201d;</span><span class="o">:</span> <span class="no">&#x201c;baz&#x201d;</span>
  <span class="o">&#125;,</span>
  <span class="no">&#x201c;key&#x201d;</span><span class="o">:</span> <span class="no">&#x201c;foo&#x201d;</span><span class="o">,</span>
  <span class="no">&#x201c;value&#x201d;</span><span class="o">:</span> <span class="mi">1</span>
<span class="o">&#125;</span></pre></div>

<p>would return</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="no">&#x201c;bar&#x201d;</span><span class="o">:</span> <span class="no">&#x201c;bar&#x201d;</span><span class="o">,</span>
  <span class="no">&#x201c;foo&#x201d;</span><span class="o">:</span> <span class="mi">1</span>
<span class="o">&#125;</span></pre></div>

<span id="shuffle"></span>

<h3 id="shuffle-items-properties"><tt>&#064;shuffle</tt> items/properties of the parameter <a href="#shuffle-items-properties" class="headerLink">#</a></h3>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>dictionary</tt> valueWithMacros that evaluates to an object or an array (required)</li>
</ul>

<p><strong>Return:</strong> for an array would produce a new array with items from <tt>dictionary</tt> but in random order. Does nothing for objects, because their properties are not ordered.</p>

<span id="merge"></span>

<h3 id="merge-arrays-objects-or"><tt>&#064;merge</tt> arrays, objects or strings <a href="#merge-arrays-objects-or" class="headerLink">#</a></h3>

<p>Allows combining multiple values of the same type into a single one.</p>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>params</tt> valueWithMacros that evaluates to an array of values of the same type (supported types are arrays, strings, and objects) (required)</li>
</ul>

<p><strong>Return:</strong></p>

<ul>
<li>if the values are strings (s1, s2, s3, ...), produces a string that is a result of concatenation of those strings, i.e. <tt>s1 + s2 + s3 + ...</tt></li>
<li>if the values are arrays (a1, a2, a3, ...), produces an array that contains items of a1 followed by items from a2, and so on. No deduplication or extra ordering is performed</li>
<li>if the values are objects (o1, o2, o3, ...), produces an object that contains a union of properties from o1, o2, o3, ... with properties of o&#123;N&#125; overriding properties of o&#123;N-1&#125;</li>
</ul>

<p><strong>Examples:</strong></p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;merge&quot;</span><span class="o">,</span>
  <span class="s2">&quot;params&quot;</span><span class="o">:</span> <span class="o">[</span> <span class="s2">&quot;foo&quot;</span><span class="o">,</span> <span class="s2">&quot;bar&quot;</span> <span class="o">]</span>
<span class="o">&#125;</span></pre></div>

<p>would result in <tt>&quot;foobar&quot;</tt>,</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;merge&quot;</span><span class="o">,</span>
  <span class="s2">&quot;params&quot;</span><span class="o">:</span> <span class="o">[</span> <span class="o">[</span> <span class="s2">&quot;a&quot;</span><span class="o">,</span> <span class="s2">&quot;b&quot;</span><span class="o">,</span> <span class="s2">&quot;c&quot;</span> <span class="o">],</span> <span class="o">[</span> <span class="s2">&quot;ab&quot;</span><span class="o">,</span> <span class="s2">&quot;c&quot;</span> <span class="o">]</span> <span class="o">]</span>
<span class="o">&#125;</span></pre></div>

<p>would result in <tt>[ &quot;a&quot;, &quot;b&quot;, &quot;c&quot;, &quot;ab&quot;, &quot;c&quot; ]</tt>,</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;merge&quot;</span><span class="o">,</span>
  <span class="s2">&quot;parmas&quot;</span><span class="o">:</span> <span class="o">[</span>
    <span class="o">&#123;</span>
      <span class="s2">&quot;foo&quot;</span><span class="o">:</span> <span class="mi">1</span>
    <span class="o">&#125;,</span>
    <span class="o">&#123;</span>
      <span class="s2">&quot;bar&quot;</span><span class="o">:</span> <span class="mi">2</span>
    <span class="o">&#125;,</span>
    <span class="o">&#123;</span>
      <span class="s2">&quot;foo&quot;</span><span class="o">:</span> <span class="mi">3</span>
    <span class="o">&#125;</span>
  <span class="o">]</span>
<span class="o">&#125;</span></pre></div>

<p>would result in</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;foo&quot;</span><span class="o">:</span> <span class="mi">3</span><span class="o">,</span>
  <span class="s2">&quot;bar&quot;</span><span class="o">:</span> <span class="mi">2</span>
<span class="o">&#125;</span></pre></div>

<span id="slice"></span>

<h3 id="obtain-subrange-of-strin">Obtain subrange of string/array/object (<tt>&#064;slice</tt>) <a href="#obtain-subrange-of-strin" class="headerLink">#</a></h3>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>dictionary</tt> valueWithMacros that evaluates to a string, array or an object (required)</li>
<li><tt>from</tt> valueWithMacros that evaluates to string if <tt>dictionary</tt> is object, int otherwise (required): starting index/key to include (inclusive)</li>
<li><tt>to</tt> valueWithMacros that evaluates to string if <tt>dictionary</tt> is object, int otherwise (required): ending index/key to include (inclusive)</li>
</ul>

<p><strong>Return:</strong></p>

<ul>
<li>if the <tt>dictionary</tt> is string, returns substring [<tt>from</tt>, <tt>to</tt>]</li>
<li>if the <tt>dictionary</tt> is an array, returns an array with items from <tt>dictionary</tt> with indexes <tt>from</tt> &lt;= index &lt;= <tt>to</tt></li>
<li>if the <tt>dictionary</tt> is an object, returns an object with properties from <tt>dictionary</tt> that have <tt>from</tt> &lt;= key &lt;= <tt>to</tt></li>
</ul>

<p><strong>Examples:</strong></p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;slice&quot;</span><span class="o">,</span>
  <span class="s2">&quot;dictionary&quot;</span><span class="o">:</span> <span class="s2">&quot;test&quot;</span><span class="o">,</span>
  <span class="s2">&quot;from&quot;</span><span class="o">:</span> <span class="mi">1</span><span class="o">,</span>
  <span class="s2">&quot;to&quot;</span><span class="o">:</span> <span class="mi">2</span>
<span class="o">&#125;</span></pre></div>

<p>would result in <tt>es</tt>,</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;slice&quot;</span><span class="o">,</span>
  <span class="s2">&quot;dictionary&quot;</span><span class="o">:</span> <span class="o">[</span> <span class="mi">4</span><span class="o">,</span> <span class="mi">6</span><span class="o">,</span> <span class="mi">3</span><span class="o">,</span> <span class="mi">9</span> <span class="o">],</span>
  <span class="s2">&quot;from&quot;</span><span class="o">:</span> <span class="mi">0</span><span class="o">,</span>
  <span class="s2">&quot;to&quot;</span><span class="o">:</span> <span class="mi">2</span>
<span class="o">&#125;</span></pre></div>

<p>would result in <tt>[ 4, 6, 3 ]</tt>,</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;slice&quot;</span><span class="o">,</span>
  <span class="s2">&quot;dictionary&quot;</span><span class="o">:</span> <span class="o">&#123;</span>
    <span class="s2">&quot;a&quot;</span><span class="o">:</span> <span class="mi">1</span><span class="o">,</span>
    <span class="s2">&quot;c&quot;</span><span class="o">:</span> <span class="mi">3</span><span class="o">,</span>
    <span class="s2">&quot;b&quot;</span><span class="o">:</span> <span class="mi">2</span><span class="o">,</span>
    <span class="s2">&quot;z&quot;</span><span class="o">:</span> <span class="mi">4</span>
  <span class="o">&#125;</span>
  <span class="s2">&quot;from&quot;</span><span class="o">:</span> <span class="s2">&quot;b&quot;</span><span class="o">,</span>
  <span class="s2">&quot;to&quot;</span><span class="o">:</span> <span class="s2">&quot;d&quot;</span>
<span class="o">&#125;</span></pre></div>

<p>would result in</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;c&quot;</span><span class="o">:</span> <span class="mi">3</span><span class="o">,</span>
  <span class="s2">&quot;b&quot;</span><span class="o">:</span> <span class="mi">2</span>
<span class="o">&#125;</span></pre></div>

<span id="sort"></span>

<h3 id="sort-an-array-of-strings"><tt>&#064;sort</tt> an array of strings/integers <a href="#sort-an-array-of-strings" class="headerLink">#</a></h3>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>dictionary</tt> valueWithMacros that evaluates to an array of integers or strings (required)</li>
</ul>

<p><strong>Return:</strong> sorted array of elements</p>

<p><strong>Examples:</strong></p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;sort&quot;</span><span class="o">,</span>
  <span class="s2">&quot;dictionary&quot;</span><span class="o">:</span> <span class="o">[</span> <span class="mi">3</span><span class="o">,</span> <span class="mi">2</span><span class="o">,</span> <span class="mi">5</span> <span class="o">]</span>
<span class="o">&#125;</span></pre></div>

<p>would result in <tt>[ 2, 3, 5 ]</tt>,</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;sort&quot;</span><span class="o">,</span>
  <span class="s2">&quot;dictionary&quot;</span><span class="o">:</span> <span class="o">[</span> <span class="s2">&quot;c&quot;</span><span class="o">,</span> <span class="s2">&quot;a&quot;</span><span class="o">,</span> <span class="s2">&quot;b&quot;</span> <span class="o">]</span>
<span class="o">&#125;</span></pre></div>

<p>would result in <tt>[ &quot;a&quot;, &quot;b&quot;, &quot;c&quot; ]</tt></p>

<span id="split"></span>

<h3 id="split-a-string-by-delime"><tt>&#064;split</tt> a string by delimeter <a href="#split-a-string-by-delime" class="headerLink">#</a></h3>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>dictionary</tt> valueWithMacros that evaluates to string (required): string that needs to be split</li>
<li><tt>delim</tt> valueWithMacros that evaluates to string (required): delimiter string</li>
</ul>

<p><strong>Result:</strong> an array of pieces of the string after splitting it, empty strings are not omitted.</p>

<p><strong>Example:</strong> <tt>&#064;split(foo::bar::baz::)</tt> would result in <tt>[ &quot;foo&quot;, &quot;bar&quot;, &quot;baz&quot;, &quot;&quot; ]</tt>.</p>

<span id="transform"></span>

<h3 id="transform-keys-and-or-it"><tt>transform</tt> keys and or items of an object or an array <a href="#transform-keys-and-or-it" class="headerLink">#</a></h3>

<p><em>This macro doesn&#039;t provide inline macroCall version.</em></p>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>dictionary</tt> valueWithMacros that evaluates to an array or an object (required): input collection for the transformation</li>
<li><tt>keyName</tt> string (optional, default: &quot;key&quot;): name of parameter to use for keys in extended contexts for <tt>itemTransform</tt> and <tt>keyTransform</tt></li>
<li><tt>itemName</tt> string (optional, default: &quot;item&quot;): name of parameter to use for values in extended contexts for <tt>itemTransform</tt> and <tt>keyTransform</tt></li>
<li><tt>keyTransform</tt> macro with extended context <tt>%key%</tt> and <tt>%item%</tt> that returns a string or an array of strings (optional): transformation macro for keys, when an array is returned, the value would be set for each key from the returned array, see example for more details</li>
<li><tt>itemTransform</tt> macro with extended context <tt>%key%</tt> and <tt>%item%</tt> that returns any value (required when <tt>dictionary</tt> is an array and optional when it&#039;s an object): macro for transforming values</li>
</ul>

<p><strong>Return:</strong></p>

<ul>
<li>if <tt>dictionary</tt> is an array, returns a new array with transformed items</li>
<li>if <tt>dictionary</tt> is an object, returns a new object with the applied transformation.</li>
</ul>

<p><strong>Examples:</strong></p>

<p>next example would multiply each item in the list by 2</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;transform&quot;</span><span class="o">,</span>
  <span class="s2">&quot;dictionary&quot;</span><span class="o">:</span> <span class="o">[</span> <span class="mi">0</span><span class="o">,</span> <span class="mi">1</span><span class="o">,</span> <span class="mi">2</span><span class="o">,</span> <span class="mi">3</span><span class="o">,</span> <span class="mi">4</span> <span class="o">],</span>
  <span class="s2">&quot;itemName&quot;</span><span class="o">:</span> <span class="s2">&quot;myCustomItemName&quot;</span><span class="o">,</span>
  <span class="s2">&quot;itemTransform&quot;</span><span class="o">:</span> <span class="s2">&quot;&#064;mul(2,%myCustomItemName%)&quot;</span>
<span class="o">&#125;</span></pre></div>

<p>the result would be <tt>[ 0, 2, 4, 6, 8 ]</tt>, next example would keep entries only with positive values and would duplicate them if the values are even</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;transform&quot;</span><span class="o">,</span>
  <span class="s2">&quot;dictionary&quot;</span><span class="o">:</span> <span class="o">&#123;</span>
    <span class="s2">&quot;foo&quot;</span><span class="o">:</span> <span class="o">-</span><span class="mi">1</span><span class="o">,</span>
    <span class="s2">&quot;bar&quot;</span><span class="o">:</span> <span class="mi">1</span><span class="o">,</span>
    <span class="s2">&quot;baz&quot;</span><span class="o">:</span> <span class="mi">2</span><span class="o">,</span>
  <span class="o">&#125;,</span>
  <span class="s2">&quot;keyName&quot;</span><span class="o">:</span> <span class="s2">&quot;myCustomKeyName&quot;</span><span class="o">,</span>
  <span class="s2">&quot;itemName&quot;</span><span class="o">:</span> <span class="s2">&quot;myCustomItemName&quot;</span><span class="o">,</span>
  <span class="s2">&quot;keyTransform&quot;</span><span class="o">:</span> <span class="o">&#123;</span>
    <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;if&quot;</span><span class="o">,</span>
    <span class="s2">&quot;condition&quot;</span><span class="o">:</span> <span class="s2">&quot;&#064;less(%myCustomItemName%,&#064;int(0))&quot;</span><span class="o">,</span>
    <span class="s2">&quot;is_true&quot;</span><span class="o">:</span> <span class="o">[],</span>
    <span class="s2">&quot;is_false&quot;</span><span class="o">:</span> <span class="o">&#123;</span>
      <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;if&quot;</span><span class="o">,</span>
      <span class="s2">&quot;condition&quot;</span><span class="o">:</span> <span class="s2">&quot;&#064;equals(&#064;int(0), &#064;mod(%myCustomItemName%, &#064;int(2)))&quot;</span><span class="o">,</span>
      <span class="s2">&quot;is_true&quot;</span><span class="o">:</span> <span class="o">[</span> <span class="s2">&quot;%myCustomKeyName%&quot;</span><span class="o">,</span> <span class="s2">&quot;%myCustomKeyName%-clone&quot;</span> <span class="o">],</span>
      <span class="s2">&quot;is_false&quot;</span><span class="o">:</span> <span class="s2">&quot;%myCustomKeyName%&quot;</span>
    <span class="o">&#125;</span>
  <span class="o">&#125;</span>
<span class="o">&#125;</span></pre></div>

<p>this would produce</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;bar&quot;</span><span class="o">:</span> <span class="mi">1</span><span class="o">,</span>
  <span class="s2">&quot;baz&quot;</span><span class="o">:</span> <span class="mi">2</span><span class="o">,</span>
  <span class="s2">&quot;baz-clone&quot;</span><span class="o">:</span> <span class="mi">2</span><span class="o">,</span>
<span class="o">&#125;</span></pre></div>

<span id="foreach"></span>

<h3 id="foreach-item-in-an-array"><tt>foreach</tt> item in an array/object <a href="#foreach-item-in-an-array" class="headerLink">#</a></h3>

<p><em>This macro doesn&#039;t provide inline macroCall version.</em></p>

<p>Provides a more generic way to execute complex query of type &quot;foreach (key, item) from &lt;from&gt; where &lt;where&gt; use &lt;use&gt; and select top &lt;top&gt; items&quot;.</p>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>key</tt> string (optional, default: &quot;key&quot;): name of parameter to use for keys in extended contexts for <tt>where</tt> and <tt>use</tt></li>
<li><tt>item</tt> string (optional, default: &quot;item&quot;): name of parameter to use for values in extended contexts for <tt>where</tt> and <tt>use</tt></li>
<li><tt>from</tt> valueWithMacros that evaluates to an array or object (required): input collection for the foreach</li>
<li><tt>where</tt> macro with extended context <tt>%key%</tt> and <tt>%item%</tt> that returns boolean (optional): select only items for which <tt>where</tt> macro returns true</li>
<li><tt>use</tt> macro with extended context <tt>%key%</tt> and <tt>%item%</tt> that always expands to the same type (either an array or an object) (optional): instead of using origional values, use result of executing <tt>use</tt> macro on key and item</li>
<li><tt>top</tt> int (optional): select first <tt>top</tt> items, unlimitted if not set</li>
<li><tt>noMatchResult</tt> any value (optional): return this value if the result is empty.</li>
</ul>

<p><strong>Return:</strong> for top <tt>top</tt> items from array/object <tt>from</tt> which satisfy <tt>where</tt> condition, merge results of expanding <tt>use</tt> macro into a single collection (array/object), if the result is empty return <tt>noMatchResult</tt>.</p>

<p><strong>Example:</strong></p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;foreach&quot;</span><span class="o">,</span>
  <span class="s2">&quot;key&quot;</span><span class="o">:</span> <span class="s2">&quot;keyRename&quot;</span><span class="o">,</span>
  <span class="s2">&quot;item&quot;</span><span class="o">:</span> <span class="s2">&quot;itemRename&quot;</span><span class="o">,</span>
  <span class="s2">&quot;from&quot;</span><span class="o">:</span> <span class="o">&#123;</span>
    <span class="s2">&quot;foo&quot;</span><span class="o">:</span> <span class="mi">1</span><span class="o">,</span>
    <span class="s2">&quot;bar&quot;</span><span class="o">:</span> <span class="mi">2</span><span class="o">,</span>
    <span class="s2">&quot;baz&quot;</span><span class="o">:</span> <span class="mi">3</span><span class="o">,</span>
    <span class="s2">&quot;abc&quot;</span><span class="o">:</span> <span class="mi">5</span>
  <span class="o">&#125;</span>
  <span class="s2">&quot;where&quot;</span><span class="o">:</span> <span class="s2">&quot;&#064;equals(&#064;mod(&#064;int(%itemRename%), &#064;int(2)), 1)&quot;</span><span class="o">,</span>
  <span class="s2">&quot;use&quot;</span><span class="o">:</span> <span class="o">[</span> <span class="s2">&quot;%keyRename%&quot;</span> <span class="o">],</span>
  <span class="s2">&quot;top&quot;</span><span class="o">:</span> <span class="mi">2</span>
<span class="o">&#125;</span></pre></div>

<p>would return a list of at most two keys of properties whose values are odd, i.e. in this case a list of two items from the set of &quot;foo&quot;, &quot;baz&quot;, &quot;abc&quot; (remember that properties inside of objects don&#039;t have any order).</p>

<span id="process"></span>

<h3 id="iterate-over-input-colle">Iterate over input collection and <tt>&#064;process</tt> each item with an accumulator <a href="#iterate-over-input-colle" class="headerLink">#</a></h3>

<p><em>This macro doesn&#039;t provide inline macroCall version.</em></p>

<p>Provides a generic way to iterate over the collection and get macro called for each item with an accumulator. Both <tt>&#064;foreach</tt> and <tt>&#064;transform</tt> can be implemented via <tt>&#064;process</tt>.</p>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>dictionary</tt> valueWithMacros that evaluates to an array or an object (required): input collection for the processing</li>
<li><tt>initialValue</tt> valueWithMacros (required): initial value of the accumulator</li>
<li><tt>keyName</tt> string (optional, default: &quot;key&quot;): name of parameter to use for keys in extended contexts for <tt>transform</tt></li>
<li><tt>itemName</tt> string (optional, default: &quot;item&quot;): name of parameter to use for values in extended contexts for <tt>transform</tt></li>
<li><tt>valueName</tt> string (optional, default: &quot;value&quot;): name of parameter to use for accumulator value in extended contexts for <tt>transform</tt></li>
<li><tt>transform</tt> macro (required): macro to be called for each item, return value of this macro would be passed to the next call of <tt>transform</tt></li>
</ul>

<p><strong>Return:</strong> the value of accumulator that was returned from the last call to <tt>transform</tt>.</p>

<p><strong>Example:</strong> next macro would iterate over the input and count how many even items are in the list.</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;process&quot;</span><span class="o">,</span>
  <span class="s2">&quot;dictionary&quot;</span><span class="o">:</span> <span class="o">[</span> <span class="mi">0</span><span class="o">,</span> <span class="mi">1</span><span class="o">,</span> <span class="mi">2</span><span class="o">,</span> <span class="mi">5</span><span class="o">,</span> <span class="mi">9</span><span class="o">,</span> <span class="mi">22</span><span class="o">,</span> <span class="mi">24</span> <span class="o">],</span>
  <span class="s2">&quot;initialValue&quot;</span><span class="o">:</span> <span class="mi">0</span><span class="o">,</span>
  <span class="s2">&quot;transform&quot;</span><span class="o">:</span> <span class="o">&#123;</span>
    <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;if&quot;</span><span class="o">,</span>
    <span class="s2">&quot;condition&quot;</span><span class="o">:</span> <span class="s2">&quot;&#064;equals(&#064;mod(%item%, 2), 0)&quot;</span><span class="o">,</span>
    <span class="s2">&quot;is_true&quot;</span><span class="o">:</span> <span class="s2">&quot;&#064;add(%value%, 1)&quot;</span><span class="o">,</span>
    <span class="s2">&quot;is_false&quot;</span><span class="o">:</span> <span class="s2">&quot;%value%&quot;</span>
  <span class="o">&#125;</span>
<span class="o">&#125;</span></pre></div>

<p>The above example would return <tt>4</tt>.</p>

<h3 id="miscellaneous-utility-ma">Miscellaneous utility macros <a href="#miscellaneous-utility-ma" class="headerLink">#</a></h3>

<span id="define"></span>

<h3 id="define-a-local-context-f"><tt>&#064;define</tt> a local context for a macro <a href="#define-a-local-context-f" class="headerLink">#</a></h3>

<p>No-op macro. Allows you to separate <tt>vars</tt> section from actual macro code.</p>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>result</tt> macro (required): result of this define</li>
</ul>

<p><strong>Return:</strong> result of calling <tt>result</tt> macro.</p>

<p><strong>Examples:</strong> assume we have next macro</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;if&quot;</span><span class="o">,</span>
  <span class="s2">&quot;vars&quot;</span><span class="o">:</span> <span class="o">&#123;</span>
    <span class="s2">&quot;A&quot;</span><span class="o">:</span> <span class="mi">0</span><span class="o">,</span>
    <span class="s2">&quot;B&quot;</span><span class="o">:</span> <span class="mi">1</span><span class="o">,</span>
    <span class="s2">&quot;C&quot;</span><span class="o">:</span> <span class="mi">2</span>
  <span class="o">&#125;,</span>
  <span class="s2">&quot;condition&quot;</span><span class="o">:</span> <span class="s2">&quot;&#064;equals(A, 0)&quot;</span><span class="o">,</span>
  <span class="s2">&quot;is_true&quot;</span><span class="o">:</span> <span class="s2">&quot;%B%&quot;</span><span class="o">,</span>
  <span class="s2">&quot;is_false&quot;</span><span class="o">:</span> <span class="s2">&quot;%C&quot;</span>
<span class="o">&#125;</span></pre></div>

<p>it can be rewritten with <tt>&#064;define</tt>:</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;define&quot;</span><span class="o">,</span>
  <span class="s2">&quot;vars&quot;</span><span class="o">:</span> <span class="o">&#123;</span>
    <span class="s2">&quot;A&quot;</span><span class="o">:</span> <span class="mi">0</span><span class="o">,</span>
    <span class="s2">&quot;B&quot;</span><span class="o">:</span> <span class="mi">1</span><span class="o">,</span>
    <span class="s2">&quot;C&quot;</span><span class="o">:</span> <span class="mi">2</span>
  <span class="o">&#125;,</span>
  <span class="s2">&quot;result&quot;</span><span class="o">:</span> <span class="o">&#123;</span>
    <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;if&quot;</span><span class="o">,</span>
    <span class="s2">&quot;condition&quot;</span><span class="o">:</span> <span class="s2">&quot;&#064;equals(A, 0)&quot;</span><span class="o">,</span>
    <span class="s2">&quot;is_true&quot;</span><span class="o">:</span> <span class="s2">&quot;%B%&quot;</span><span class="o">,</span>
    <span class="s2">&quot;is_false&quot;</span><span class="o">:</span> <span class="s2">&quot;%C&quot;</span>
  <span class="o">&#125;</span>
<span class="o">&#125;</span></pre></div>

<span id="range"></span>

<h3 id="generate-an-array-with-a">Generate an array with a <tt>&#064;range</tt> if integers <a href="#generate-an-array-with-a" class="headerLink">#</a></h3>

<p>Utility macro that generates a range of integers (from &lt;= integer &lt;= to).</p>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>from</tt> valueWithMacros that evaluates to integer (required): start of the range</li>
<li><tt>to</tt> valueWithMacros that evaluates to integer (required): end of the range</li>
</ul>

<p><strong>Return:</strong> an array with integers from the range (from &lt;= integer &lt;= to).</p>

<p><strong>Example:</strong> <tt>&#064;range(3, 5)</tt> would produce an array <tt>[3, 4, 5]</tt>; <tt>&#064;range(3, 2)</tt> would produce <tt>[]</tt>.</p>

<span id="defined"></span>

<h3 id="check-if-the-macro-param">Check if the macro/parameter/constant is <tt>&#064;defined</tt> <a href="#check-if-the-macro-param" class="headerLink">#</a></h3>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>name</tt> string (required): name of macro/parameter/constant to check for existance</li>
</ul>

<p><strong>Return:</strong> true if the macro/parameter/constant exist in current context, false otherwise</p>

<p><strong>Example:</strong></p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;define&quot;</span><span class="o">,</span>
  <span class="s2">&quot;vars&quot;</span><span class="o">:</span> <span class="o">&#123;</span>
    <span class="s2">&quot;A&quot;</span><span class="o">:</span> <span class="s2">&quot;B&quot;</span>
  <span class="o">&#125;</span>
  <span class="s2">&quot;result&quot;</span><span class="o">:</span> <span class="o">&#123;</span>
    <span class="s2">&quot;a-exists&quot;</span><span class="o">:</span> <span class="s2">&quot;&#064;defined(A)&quot;</span><span class="o">,</span>
    <span class="s2">&quot;non-existing-exist&quot;</span><span class="o">:</span> <span class="s2">&quot;&#064;defined(non-existing)&quot;</span>
  <span class="o">&#125;</span>
<span class="o">&#125;</span></pre></div>

<p>would be preprocessed to</p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;a-exists&quot;</span><span class="o">:</span> <span class="kc">true</span><span class="o">,</span>
  <span class="s2">&quot;non-existing-exist&quot;</span><span class="o">:</span> <span class="kc">false</span>
<span class="o">&#125;</span></pre></div>

<p>unless &quot;non-existing&quot; was defined in some outer context.</p>

<span id="fail"></span>

<h3 id="fail-preprocessing"><tt>&#064;fail</tt> preprocessing <a href="#fail-preprocessing" class="headerLink">#</a></h3>

<p>Explicitly direct to fail preprocessor with some message. Useful when some invalid input was provided and you need to signal that things are broken.</p>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>msg</tt> valueWithMacros that evaluates to a string (required): message to emit as a failure reason</li>
</ul>

<p><strong>Return:</strong> no return, preprocessing is terminated with the given failure reason.</p>

<p><strong>Example:</strong> <tt>&#064;fail(failure)</tt> will terminate with the message &quot;failure&quot;.</p>

<span id="isLocalIp"></span>

<h3 id="check-if-an-ip-address-i">Check if an IP address <tt>&#064;isLocalIp</tt> <a href="#check-if-an-ip-address-i" class="headerLink">#</a></h3>

<p>Checks if the parameter is an IP address that belongs to the current host.</p>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>ip</tt> valueWithMacros that evaluates to a string (required): IP address to check, but can be any string</li>
</ul>

<p><strong>Return:</strong> true if the supplied <tt>ip</tt> is a valid IP address and compares true to one of local addresses, false otherwise.</p>

<p><strong>Examples:</strong> <tt>&#064;isLocalIp(::1)</tt> would evaluate to true, while <tt>&#064;isLocalIp(blah)</tt> would evaluate to false.</p>

<span id="hash"></span>

<h3 id="compute-hash-of-an-integ">Compute <tt>&#064;hash</tt> of an integer or a string <a href="#compute-hash-of-an-integ" class="headerLink">#</a></h3>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>value</tt> valueWithMacros that evaluates to an integer or a string (required): value to compute hash for</li>
</ul>

<p><strong>Return:</strong> hash of <tt>value</tt>.</p>

<p><strong>Examples:</strong> <tt>&#064;hash(abc)</tt>, <tt>&#064;hash(&#064;int(123))</tt>.</p>

<span id="weightedHash"></span>

<h3 id="compute-weightedhash-of">Compute <tt>&#064;weightedHash</tt> of an integer or a string <a href="#compute-weightedhash-of" class="headerLink">#</a></h3>

<p>Compute Rendezvous Hash for given input and a dictionary of choices.</p>

<p><strong>Parameters:</strong></p>

<ul>
<li><tt>dictionary</tt> valueWithMacros that evaluates to an object with floating point properties (required): set of choices to choose from</li>
<li><tt>key</tt> valueWithMacros that evaluates to an integer or a string (required): value to hash</li>
</ul>

<p><strong>Return:</strong> some key from <tt>dictionary</tt> object.</p>

<p><strong>Example:</strong></p>

<div class="remarkup-code-block" data-code-lang="php"><pre class="remarkup-code"><span class="o">&#123;</span>
  <span class="s2">&quot;type&quot;</span><span class="o">:</span> <span class="s2">&quot;weightedHash&quot;</span><span class="o">,</span>
  <span class="s2">&quot;dictionary&quot;</span><span class="o">:</span> <span class="o">&#123;</span>
    <span class="s2">&quot;a&quot;</span><span class="o">:</span> <span class="mf">0.0</span><span class="o">,</span>
    <span class="s2">&quot;b&quot;</span><span class="o">:</span> <span class="mf">1.0</span>
  <span class="o">&#125;,</span>
  <span class="s2">&quot;key&quot;</span><span class="o">:</span> <span class="mi">5</span>
<span class="o">&#125;</span></pre></div>

<p>would return <tt>&quot;b&quot;</tt>.</p>
