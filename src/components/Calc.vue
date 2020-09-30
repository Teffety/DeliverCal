<template>
  <div class="calc">
    <p class="calc-error" v-if="error"> Недопустимое выражение</p>
    <div class="calc-value" :class="[error && '-error-', isFocus && '-focus-']" >
      <p class="calc-value_old">{{ summValue }}</p>
      <input class="calc-input" 
        v-model="value" 
        @keyup="keyup" 
        @focus="isFocus = true"
        @blur="isFocus = false"
        placeholder="0"
        type="text" />
    </div>
    <div class="calc-buttons">
      <div class="calc-buttons_numbers">
        <div class="calc-buttons_number"
          v-for="i in buttons" :key="i" 
          :class="i === '' && '-empty-'"
          @click="click(i)"
        >
          <p class="calc-buttons_text">
            {{i}}
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data()
  {
    return {
      value: 0,
      summValue: '',
      scobe:'',
      error: false,
      isFocus: false
    }
  },
  computed:
  {
    // формирование массива для отрисовки кнопок
    buttons()
    {
      return 'CE C ^ / 7 8 9 * 4 5 6 - 1 2 3 + 0 ( ) ='.split(' ')
    },
    // набор цифр
    numbers()
    {
      return '1 2 3 4 5 6 7 8 9 0'
    },
    // набор используемых символов
    symbols()
    {
      return 'CE C ^ / * - + ( ) ='
    }
  },
  watch:
  {
    value()
    {
      // Создание float числа для ввода
      let valueIdx = this.value.toString().split('').filter( el => el == '.').length;
      let val = parseFloat(this.value)
      let vIdx = val.toString().split('').filter( el => el == '.').length;

      if( isNaN(val) )
        this.value = 0
      else if(valueIdx === 1 && vIdx === 0)
        return 
      else 
        this.value = val
    }
  },
  methods: 
  {
    // Функция при нажатие клавишы клавиатуры
    keyup( e )
    {
      if ( (e.key).match(/[0-9\/*\-+\(\)^=]|Backspace|Enter|Escape/) )
        this.setValue(e.key)
    },
    // Функция при нажатие на кнопку калькулятора
    click( key )
    {
      if( Number( key ) || key === '0')
        this.value += key
      else
        this.setValue( key )
    },
    // условия отрисовки значения
    setValue( key )
    { 
      let val = this.summValue.toString()
      val = val[ val.length - 1 ];
      let str = this.value + key

      if( this.symbols.includes( key ) && !( key === 'Enter' || key === '=' || key === 'CE' || key === 'Escape' || key === 'C' ))
      {
        if( val === key && !this.value )
          return 
        else if ( !this.value && this.numbers.includes( val ) &&  ( key !== '(' || key !== ')' ) )
          str = key;
        else if ( !this.value && this.symbols.includes( val ) &&   key !== '(' && key !== ')'  )
          str = key;
        else if ( val === undefined && this.value !== 0 && ( key === ')' || key === '(' || key === '-' ) )
          str = key + this.value
        else if ( this.symbols.includes( val ) && ( key === '(' ) )
          str = key + this.value;
        else if ( this.symbols.includes( val ) && key === ')')
          str = this.value + key;
        else if ( this.numbers.includes( val ) && ( key !== '(' || key !== ')' ) )
          str = key + this.value;
      
        this.summValue+= str
        this.value = ''
      }

      if ( key === 'Enter' || key === '=' ) 
        this.enteryValue()

      if( key === 'CE' || key === 'Escape' || key === 'C' )
        this.clearValue( key )
    },
    // Очищение поля
    clearValue( key )
    {
      if( key === 'C' )
        this.value = '';
      else
      {
        this.summValue = '';
        this.value = '';
      }
    },
    // Презапуск подсчета
    enteryValue()
    {
      if( this.summValue.length === 0 || !this.summValue[ this.summValue.length - 1])
        return

      this.summValue = this.summValue + this.value
      this.value = ''
      this.calValue();
    },
    // Подсчет с проверкой
    calValue()
    {
      this.error = false;
      let str  = this.summValue.replace(/\^/g, '**');
      let func;
      try {
        func = new Function( 'return ' + str )();
      } catch ( error ) {
        func = ''
        this.value = ''
      }
      if( !func )
        this.error = true
      this.summValue = func
    }
  }
}
</script>