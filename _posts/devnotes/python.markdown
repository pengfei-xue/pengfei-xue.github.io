## string format

    content = u'''
        <table><tbody>
          <tr>
            <td> 响应时间平均值 </td>
            <td> {mean:.4f} </td>
          </tr>
          <tr>
            <td> 响应时间75% </td>
            <td> {seventy_five:.4f} </td>
          </tr>
          <tr>
            <td> 响应时间95% </td>
            <td> {ninety_five:.4f} </td>
          </tr>
          <tr>
            <td> 可用性 </td>
            <td> {ava} </td>
          </tr>
        </tbody></table>
    '''.format(\*\*{
        'mean': df.mean()[0],
        'seventy_five': df.quantile(.75)[0],
        'ninety_five': df.quantile(.95)[0],
        'ava': ava
    })

[Using % and .format() for great good!](https://pyformat.info/)


## How can I use Python to get the system hostname?

    import socket
    print(socket.gethostname())

    import platform
    platform.node()

http://stackoverflow.com/questions/4271740/how-can-i-use-python-to-get-the-system-hostname


## Python: defaultdict of defaultdict?

    defaultdict(lambda : defaultdict(int))

[Python: defaultdict of defaultdict](http://stackoverflow.com/questions/5029934/python-defaultdict-of-defaultdict)


## get random string

Generating strings from (for example) lowercase characters:

    import random, string

    def randomword(length):
       return ''.join(random.choice(string.lowercase) for i in range(length))

    Results:
    >>> randomword(10)
    'vxnxikmhdc'
    >>> randomword(10)
    'ytqhdohksy'

[get random string](http://stackoverflow.com/questions/2030053/random-strings-in-python)
